# Linux-asennus

Chloros jaetaan Linuxille `.deb`-paketteina, jotka asentavat CLI:n, taustapalvelimen ja Python-SDK:n (järjestelmälle `python3` sekä versioon sopivan wheel-tiedoston virtualenvs-ympäristöjä varten).

Pakettitiedostojen nimissä on versio ja arkkitehtuuri: `chloros_1.3.0_amd64.deb` x86_64:lle, ja `chloros_1.3.0_arm64_jp6.deb` arm64:lle (NVIDIA Jetson, JetPack 6). Korvaa alla olevissa komennoissa tiedosto, jonka olet itse ladannut.

{% hint style="info" %}
**Uutta versiossa 1.3.0: ohjelmat toimitetaan pakkaamattomina.** Taustaprosessi ja CLI asennetaan kansioina, `/usr/lib/chloros/backend/` ja `/usr/lib/chloros/cli/`, yksittäisten itsepurkautuvien tiedostojen sijaan. Ne eivät enää pura itseään väliaikaiseen kansioon jokaisen käynnistyksen yhteydessä, joten ne käynnistyvät paljon nopeammin (x86_64-alustalla taustapalvelu käytti aiemmin noin 100 sekuntia purkamiseen jokaisen käynnistyksen yhteydessä) eivätkä ne enää vaadi useita gigatavuja vapaata tilaa kansiossa `/tmp`. Myös amd64-versio `.deb` on kooltaan noin puolet entisestä.
{% endhint %}

***

## Linux-amd64 (x86_64)

### Järjestelmävaatimukset

| Vaatimus | Vähimmäisvaatimus | Suositeltu |
| --- | --- | --- |
| **Jakelu** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Prosessori** | x86_64 (Intel/AMD) | Intel Core i7 tai parempi |
| **Muisti (RAM)** | 8 GB | 16 GB tai enemmän |
| **Näytönohjain** | Ei tarvita (käsitellään prosessorilla) | NVIDIA-näytönohjain, jossa vähintään 4 GB VRAM-muistia (vähintään 12 GB avaa `GPU_PARALLEL`-ominaisuuden, vähintään 7 GB pitää Texture Aware -ominaisuuden pois käytöstä yksittäiskuvapolulla) |
| **Tallennustila** | 2 GB vapaata tilaa | SSD, jossa vähintään 10 GB vapaata tilaa |
| **Python** | Python 3.7 tai uudempi (SDK:lle) | Python 3.10 tai uudempi |

> **Ubuntu 20.04:ää ja Debian 11:tä ei tueta.** `.deb`:n riippuvuuslista on
> johdettu siitä, mihin Chloros-taustakomponentti tosiasiallisesti linkittyy, ja siihen sisältyy
> `libc6`, joka on uudempi kuin focal- ja bullseye-jakeluissa mukana oleva glibc 2.31, joten `apt` hylkää
> asennuksen suoraan sen sijaan, että antaisi sen epäonnistua myöhemmin suorituksen aikana.

### Asennus

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
```

{% hint style="info" %}
**Käytä `apt install ./…`:ää, älä `dpkg -i`:ää.** `apt` ratkaisee paketin riippuvuudet ja asentaa myös sen suosittelemat: Python-SDK:n vaatimukset, `python3-numpy` ja `python3-requests`. `dpkg -i` ei asenna kumpaakaan. CLI-käyttöliittymä ja taustapalvelu ovat itsenäisiä ja toimivat ilman niitä, mutta `import chloros_sdk` tarvitsee niitä; asennuksen tuloste mainitsee puuttuvat osat. Säilytä `./`, jotta `apt` asentaa tiedoston sen sijaan, että se etsisi kyseisellä nimellä varustettua pakettia. Paketti määrittelee myös `libgl1`:n, jonka mukana toimitettu OpenCV lataa ja joka saattaa puuttua minimaalisesta palvelinimageesta.
{% endhint %}

Varmista asennus:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo apt install ./chloros_1.3.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block ("chloros_sdk 1.3.0 for python3" and the "In a virtualenv: pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl" line), any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend@<user>.service" hint -->

***

## Linux arm64 (NVIDIA Jetson)

### Järjestelmävaatimukset

| Vaatimus | Vähimmäisvaatimus | Suositeltu |
| --- | --- | --- |
| **Alusta** | NVIDIA Jetson ja JetPack 6 | Jetson Orin NX 16 GB tai AGX Orin |
| **JetPack** | JetPack 6.x | Uusin JetPack 6 |
| **Muisti (RAM)** | 8 GB (jaettu GPU/CPU) | 16 GB+ jaettu (12 GB+ on rinnakkaisprosessoinnin kynnysarvo) |
| **Tallennustila** | 2 GB vapaata tilaa | NVMe SSD, jossa vähintään 10 GB vapaata tilaa |
| **Python** | Python 3.7+ (SDK:lle) | Python 3.10+ |

### Asennus

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Sama rakenne kuin amd64-versiossa `.deb` (sisältää Python-SDK:n, vastaanottopuskurit ja taustapalvelut), Jetson Orin / Orin NX / Orin Nano -laitteille **JetPack 6**:ssa (L4T R36). Sen PyTorch on JetPack 6:n CUDA-rakennus Orinille, joten GPU-käsittely, mukaan lukien Texture Aware -debayer, suoritetaan Jetsonin GPU:lla. Paketti ei sisällä itse CUDAa: se käyttää CUDA- ja cuDNN-kirjastoja, jotka JetPack 6 asentaa (`sudo apt install nvidia-jetpack` kuvassa, josta ne puuttuvat). Jos CUDA-GPU:ta ei voida käyttää, käsittely tapahtuu CPU:lla ja `--debayer texture-aware` käyttää Standard-debayeria (taustaprosessi kirjaa yhden `[DEBAYER] … needs a CUDA GPU …`-rivin, jossa kerrotaan syy). Jetsonin muistiin, lämpötilaan ja kenttäkäyttöön liittyvistä ominaisuuksista on lisätietoja [NVIDIA Jetson -oppaassa](nvidia-jetson-guide.md).

***

## Python-SDK:n asennus (kaikki Linux-järjestelmät)**`.deb` asentaa SDK:n puolestasi.** Molemmat paketit (amd64 ja Jetson) asentavat sen järjestelmään `python3`, joten `import chloros_sdk` toimii heti asennuksen jälkeen ilman pip:iä tai verkkoyhteyttä. Sen paketit (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` sekä projektin lataajat) sijoitetaan `/usr/lib/python3/dist-packages`:ään, jonka omistaa kyseinen paketti: päivitys korvaa ne, ja paketin poistaminen poistaa ne. Versio vastaa aina asentamaasi CLI:tä ja taustapalvelinta.

SDK:n omat vaatimukset, `python3-numpy` ja `python3-requests`, ovat **suositeltavia, eivät pakollisia**: `sudo apt install ./chloros_<version>_<arch>.deb` asentaa ne, `dpkg -i` ei. Asennuksen tuloste ilmoittaa SDK-version, jonka `python3` tuo mukanaan, mahdolliset puuttuvat vaatimukset sekä kaikki `chloros_sdk`-tiedostot, jotka sijaitsevat muualla `python3`:n polulla ja peittävät paketin kopion (kuten aiemmin `pip install --user`:n avulla asennettu versio).**Virtualenv-ympäristössä**, joka ei näe järjestelmän paketteja, asenna mukana toimitettu wheel-tiedosto siihen (täysin offline-tilassa):

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

**PyPI:stä** — vain koneelle, johon ei ole asennettu Chloros-pakettia ja jonka SDK-ympäristö ohjaa toisella koneella sijaitsevaa taustapalvelinta (katso [Etätaustapalvelintila](../reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

Valinnaiset lisäosat pip-asennusta varten (virtualenv tai PyPI):

| Lisäosa | Komento | Lisää |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` reaaliaikaista edistymisen suoratoistoa varten |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` BLE (DAQ-M) -siirtoa varten |

Varmista SDK:

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Python-SDK kommunikoi `.deb`:n asentaman taustapalvelimen kanssa paikallisen HTTP-API:n (`http://127.0.0.1:5000`) kautta. `ChlorosLocal` ja yhteysaputoiminnot käynnistävät sen automaattisesti tarvittaessa; tallennetun projektin `connect_all()` ei koskaan tee niin, joten ajattomalla koneella taustapalvelu on ajettava [palveluna](#always-on-ptp-for-headless-hosts). Käytä aina kirjaimellista IPv4-osoitetta `localhost`:n sijaan — `localhost` voi johtaa `::1`:ään ja maksaa noin kaksi sekuntia pyyntöä kohti.
{% endhint %}

***

## Ensimmäinen asennus

### 1. Kirjaudu sisään

CLI- ja SDK-käyttö edellyttää maksullista Chloros+-tasoa (**Copper** tai korkeampi), mikä valvotaan palvelinpuolella: uloskirjautunut käyttäjä saa osoitteen `401 AUTH_REQUIRED`, ja ilmaistason (Iron) käyttäjä saa osoitteen `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Kirjautumistiedot tallennetaan välimuistiin `~/.chloros/user_session.json`:ssä.

{% hint style="info" %}
**Päivitykset säilyttävät kirjautumistietosi; poistaminen tyhjentää ne.** Uudemman version (`.deb`) asentaminen olemassa olevan asennuksen päälle (myös `chloros-cli update`:n kautta) säilyttää `~/.chloros/user_session.json`:n ja välimuistiin tallennetun lisenssin, joten komentorivipohjaiset (CLI) ja SDK-tehtävät toimivat edelleen. Paketin (`sudo apt remove chloros`) poistaminen tyhjentää tallennetut kirjautumistiedot ja lisenssivälimuistin kaikilta koneen käyttäjiltä, joten uusi asennus vaatii jälleen `chloros-cli login`:n.
{% endhint %}

### 2. Tarkista lisenssin tila

```bash
chloros-cli status
```

`chloros-cli status` toimii kaikilla tasoilla (myös ilmaisella), joten voit aina selvittää, miksi pääsy on tai ei ole käytettävissä.

### 3. Suorita järjestelmän vianmääritys

```bash
chloros-cli selftest
```

Seitsemän tarkistusta suoritetaan järjestyksessä, ja komento palauttaa nollasta poikkeavan arvon, jos jokin niistä epäonnistuu:

| # | Tarkistus | Mitä se todistaa |
| --- | --- | --- |
| 1 | **Versio** | CLI ilmoittaa versionsa (`v1.3.0`). |
| 2 | **Portti käytettävissä** | Portti 5000 on vapaa *tai* siihen on jo vastannut toimiva Chloros-taustapalvelin (mikä lasketaan hyväksytyksi). |
| 3 | **Taustapalvelimen käynnistys** | Taustapalvelimen binääritiedosto käynnistyy. |
| 4 | **API-testi (`/api/test`)** | Taustapalvelin vastaa `status: ok`. |
| 5 | **Järjestelmätiedot** | Tulostaa `GPU: <name>, CUDA: <bool>, PyTorch: <version>` kohdasta `/api/system-info`. |
| 6 | **Denoiser-mallit** | Löytää `*.pth.enc`-mallit (Linuxissa: `/usr/lib/chloros/models`). |
| 7 | **CUDA + kohinanpoistaja**| Texture Aware on todellakin käyttökelpoinen — vaatii CUDA:n**ja** vähintään yhden mallitiedoston. |

Suoritus päättyy `N/7 checks passed`:ään, jossa luetellaan mahdolliset epäonnistumiset nimeltä.

### 4. Käsittele ensimmäinen aineistosi

```bash
chloros-cli process ~/datasets/flight001
```

***

## Tiedostot ja hakemistot

### Käyttäjäkohtainen

Chloros säilyttää tunnistetietonsa ja CLI-asetuksensa yhdessä alustojen välisessä hakemistossa, **`~/.chloros/`** (Windowsissa `%USERPROFILE%\.chloros\`). Kaksi Linux-kohtaista välimuistia noudattaa sen sijaan XDG-käytäntöjä — nämä käyttävät `XDG_CONFIG_HOME` / `XDG_CACHE_HOME`-polkuja, kun ne on määritetty.

| Polku | Käyttötarkoitus |
| --- | --- |
| `~/.chloros/user_session.json` | `chloros-cli login`:n kirjoittama kirjautumissession välimuisti (säilyy pakettipäivitysten yli; tyhjennetään, kun paketti poistetaan) |
| `~/.chloros/working_directory.txt` | Oletusprojektikansion ohitus (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | CLI-kieliasetus (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Kieli-asetus jaetaan Windows-käyttöliittymän kanssa — tässä `language`-asetus on etusijalla `cli_language.json`-asetukseen nähden |
| `~/.chloros/update_cache.json` | Yhden tunnin välimuisti Linux/Jetsonin käynnistyspäivitysten tarkistusta varten |
| `~/.chloros/backend.log` | Taustapalvelimen loki, kun taustapalvelin käynnistettiin CLI:n kautta |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Välimuistissa olevat kamerakohtaiset LATTICE-kalibrointipaketit, indeksoitu sarjanumeron ja nipun hajautusarvon perusteella |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Valinnaiset käyttäjän tekemät ohitukset DAQ-kapasiteettikorjausprofiileille |
| `~/.config/chloros/system_config.json` | Dynamic Compute Adaptation -toiminnosta välimuistiin tallennettu laitteistoprofiili — poista se, jos haluat pakottaa uuden laitteistotunnistuksen |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Taustapalvelimen lokit, yksi tiedosto kutakin käynnistystä kohti |
| `~/Documents/MAPIR/Chloros Projects/` | Oletusprojektikansio, kun ylikirjoitusta ei ole määritetty |

### Koko järjestelmän laajuinen

| Polku | Tarkoitus |
| --- | --- |
| `/usr/bin/chloros-cli` | Kääreskripti — asettaa `LD_LIBRARY_PATH`:n mukana toimitettujen natiivikirjastojen kohdalle (ja `TMPDIR`:n arvoksi `/mnt/ssd/tmp`:n, jos kyseinen kansio on olemassa), minkä jälkeen suorittaa varsinaisen binääritiedoston |
| `/usr/bin/chloros-backend` | Kääreskripti — sama taustaprosessille, jonka se merkitsee asennetuksi tuotantoversioksi |
| `/usr/lib/chloros/cli/`, `/usr/lib/chloros/backend/` | Käännetyt CLI- ja taustaprosessiohjelmat, asennetaan purkuna (uusi ominaisuus versiossa 1.3.0); ohjelmat ovat `cli/chloros-cli` ja `backend/chloros-backend` |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Kaksiriviset käynnistysskriptit, jotka käynnistävät yllä mainitut varsinaiset ohjelmat — säilytetään näissä poluissa, koska SDK, CLI ja vanhemmat skriptit etsivät niitä täältä |
| `/usr/lib/chloros/arena_runtime/` | LATTICE-kameroiden edellyttämä Arena-SDK-ajoympäristö |
| `/usr/lib/chloros/models/*.pth.enc` | Texture Aware -debayerin käyttämät salatut kohinanpoistomallit |
| `/usr/lib/python3/dist-packages/chloros_sdk/` (sekä `lattice_sdk`, `lattice_cam`, `arena_api`, projektin latausohjelmat) | Python-SDK, asennettu järjestelmään `python3` ja paketin omistama |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Juuri tähän rakennukseen sopiva Python-SDK-wheel virtuaaliympäristöä varten |
| `/usr/lib/chloros/exiftool` | Mukana toimitettu ExifTool-käynnistysohjelma (symlinkitetty `/usr/local/bin/exiftool`:ään vain, jos järjestelmässä ei ole exiftool-työkalua). Versiosta 1.3.0 lähtien ExifTool-skripti toimitetaan omalla Perl-kirjastollaan tiedostossa `/usr/lib/chloros/exiftool-dist/`, joten se toimii ilman järjestelmän `libimage-exiftool-perl`-pakettia |
| `/etc/chloros/update.conf` | `chloros-cli update` lukee päivityskanavan asetukset |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Asettaa `net.ipv4.ip_unprivileged_port_start = 319`:n siten, että taustaprosessi voi sitoa PTP-portit ilman pääkäyttäjän oikeuksia |
| `/etc/sysctl.d/60-chloros-gige.conf` | Suurentavat ytimen socket-vastaanottopuskureita (`net.core.rmem_max`, `net.core.rmem_default`) 32 Mt:iin LATTICE-kameravirtoja varten ja ottaa ne käyttöön asennuksen yhteydessä (jos arvo on jo asetettu suuremmaksi, sitä ei muuteta). Ytimen oletusasetuksilla kamerapaketit hylätään ytimessä, jolloin kuvasarja menettää kehyksiä ja kehysnopeutta; `chloros-cli lattice network` ilmoittaa puskureiden tilasta, kun ne ovat vähissä. Katso [Linux-isäntäkoneet — socket-vastaanottopuskuri](../reference/cli-reference.md#linux-hosts--socket-receive-buffers) |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Ohjaa dynaamisen lataajan kohteeksi `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Myöntää kirjautuneelle käyttäjälle pääsyn DAQ-U USB-sarjaliitännän (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend@.service` | Valinnainen aina päällä oleva taustapalvelu, joka suorittaa taustapalvelun nimettynä käyttäjänä (asennettu, **ei käytössä**) — tämä on se, joka on otettava käyttöön, katso [Always-On PTP](#always-on-ptp-for-headless-hosts) |
| `/lib/systemd/system/chloros-backend.service` | Sama palvelu, joka suorittaa taustapalvelua **root**-käyttäjänä (asennettu,**ei käytössä**), pidetään mukana, jotta kone, jolla se on otettu käyttöön, pysyy käynnissä |
| `/var/log/chloros/` | Root-palvelun yksityiskohtainen loki, jota `adm`-ryhmä voi lukea (poistetaan, kun paketti poistetaan kokonaan) |
| `/usr/share/applications/chloros-cli.desktop` | ”Chloros CLI” -sovellusvalikkokohta, joka avaa terminaalin |

**Palomuuri (uusi versio 1.3.0).** Jos `ufw` tai `firewalld` on aktiivinen asennuksen aikana, paketti avaa UDP-portit `5002` (DAQ-E-tiedot), `5353` (mDNS-löytö), `319` ja `320` (PTP), ja poistaa nämä säännöt, kun paketti poistetaan. Jos palomuuri on asennettu mutta ei aktiivinen, se ei lisää mitään ja tulostaa sallittavat portit siltä varalta, että otat sen käyttöön myöhemmin. Se ei avaa TCP-portteja: DAQ-E-laiteohjelmiston päivitys, jossa anturi muodostaa yhteyden takaisin tähän tietokoneeseen TCP:n kautta kuvan keräämiseksi, epäonnistuu virheen `… device never connected back on TCP <port> …` vuoksi, kun aktiivinen palomuuri estää kyseisen yhteyden.

## Taustaprosessin suoritustiedoston sijainti

CLI ja SDK tunnistavat taustaprosessin automaattisesti:

| Komponentti | Polku |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Taustapalvelu | `/usr/lib/chloros/chloros-backend` (käynnistysohjelma sovellukselle `/usr/lib/chloros/backend/chloros-backend`) |

Taustapalvelimen polku voidaan ohittaa CLI-lipulla `--backend-exe` tai SDK-konstruktoriparametrilla `backend_exe`, ja portin `--port`:llä (oletusarvo `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` ohjaa **`lattice`**-,**`project`**ja**`daq pool-*`**-komentoperheisiin etäisessä taustapalvelimessa. Ydinkomennot (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) jättävät sen tarkoituksella huomiotta ja kohdistuvat aina komentoon `http://127.0.0.1:<port>`.
{% endhint %}

***

## LATTICE-kamerat ja DAQ-valosensorit Linuxissa

Kaikki live-hardware-komentoryhmät toimivat Linuxissa (amd64 ja Jetson):

* **`chloros-cli lattice`** — tunnistaa, yhdistää, määrittää ja tallentaa LATTICE-kameroista ja synkronoiduista matriiseista. `.deb` sisältää tarvittavan Arena-SDK-ajoympäristön ja rekisteröi sen dynaamiseen lataajaan.
* **`chloros-cli daq pool-*`** — yhdistä DAQ-U/M/E-valosensorit taustapoolin kautta, lähetä kalibroituja spektrejä, tallenna `.daq`-tiedostoja ja vaihda cap-profiileja sekä muuta DAQ-E:n verkkoasetuksia, käynnistä se uudelleen ja päivitä sen laiteohjelmisto. `chloros-cli daq discover` etsii antureita, joihin voit muodostaa yhteyden, ja `chloros-cli daq --help` luettelee `pool-*`-alikomennot.
* **`chloros-cli project`** — aja tallennettua projektia (sen kameroita, antureita ja käsittelyasetuksia) ilman käyttöliittymää.
* **`chloros-cli time-sync`** — tarkista PTP-grandmaster, jota Chloros-taustapalvelu käyttää LATTICE-kameroille ja DAQ-E-antureille.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` on pakollinen komento komentojen `pool-latest`, `pool-stream`, `pool-record` ja `pool-set-cap`; `pool-list` näyttää poolissa tällä hetkellä olevat tunnukset.

{% hint style="info" %}
**`--eth-host` ohittaa tunnistuksen.** `pool-connect --eth` löytää DAQ-E:n mDNS:n avulla, ja sen ARP-varajärjestelmä toimii tyhjästä ARP-välimuistista; antamalla anturin IP-osoitteen tai isäntänimen komennolla `--eth-host` muodostetaan suora yhteys siihen.
{% endhint %}

**DAQ-U:n sarjaliikenneoikeudet** hoidetaan asennetun udev-säännön avulla (`uaccess` + ryhmä `dialout`). Jos jo liitetty anturi ei ole käytettävissä, lataa säännöt uudelleen tai liitä se uudelleen:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Katso [CLI-ohje](../CLI.md) saadaksesi täydellisen komentovalikoiman.

### Aina päällä oleva PTP päätä vailla oleville isäntäkoneille

Paketti asentaa kaksi systemd-palvelua taustajärjestelmälle, joista kumpikaan ei ole käytössä. Päätä vailla olevassa Jetson-laitteessa tai palvelimessa, jonka on pidettävä PTP-aikasynkronointi jatkuvasti käynnissä DAQ-E-antureita ja LATTICE-kameroita varten, tai jossa SDK-skriptit yhdistävät tallennettuja projekteja (`connect_all()` ei koskaan käynnistä taustapalvelua), ota palvelu käyttöön **omalla käyttäjätunnuksellasi**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` suorittaa taustapalvelun kyseisenä käyttäjänä: sen kirjoittamat tallenteet kuuluvat kyseiselle käyttäjälle, sen yksityiskohtainen loki on kyseisen käyttäjän `~/.cache/chloros/logs`-tiedostossa, ja se käyttää kyseisen käyttäjän `chloros-cli login` (`~/.chloros`), joten kirjaudu ensin kerran kyseisenä käyttäjänä sisään. Paketin asennus tulostaa tämän komennon, johon on täytetty käyttäjänimesi.

Ilman palvelua taustaprosessi (ja sen mukana PTP-grandmaster) toimii vain, kun jokin on käynnistänyt sen — joko `chloros-cli`-komento tai SDK-skripti.

Laite sitoo taustaprosessin yksikön sisäisiin `127.0.0.1:5000` (yksikön sisäiset ympäristöasetukset `CHLOROS_HOST` / `CHLOROS_PORT`; ohita asetukset komennolla `sudo systemctl edit chloros-backend@.service`) ja käynnistää sen uudelleen 5 sekunnin kuluttua, jos se kaatuu.

**Root-palvelu.**`chloros-backend.service` suorittaa taustapalvelua**root**-käyttäjänä. Se on edelleen asennettuna, jotta kone, jolla se on otettu käyttöön, pysyy käynnissä, mutta kaikki, mitä root-taustapalvelu kirjoittaa, kuuluu rootille, joten käyttäjä ei voi poistaa omia tallenteitaan; sen loki on tiedostossa `/var/log/chloros`, jota `adm`-ryhmä voi lukea. Voit vaihtaa omaan käyttäjätunnukseesi:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

**Palvelun taustaprosessi on jaettu, eikä asiakas voi koskaan pysäyttää sitä.** `chloros-cli`- ja SDK-skriptit käyttävät sitä sellaisenaan: `process` toimii sen päällä ja lataa oman projektinsa edellisen tilalle. Siinä missä CLI-komento muuten käynnistäisi taustapalvelun uudelleen (`--restart` tai `process`, jonka tuonti ei löytänyt mitään), se nimeää yksikön, tulostaa `sudo systemctl restart <unit>` ja lopettaa `1`:n, koska tällä tavalla pysäytetty taustapalvelu pysyisi pois päältä. Käynnistä se uudelleen itse komennolla `sudo systemctl restart chloros-backend@$USER.service`. Palvelun taustaprosessi ei koskaan pysäytä itseään, kun se on käyttämättömänä, ja `chloros-cli backend stop` kieltäytyy siitä ja nimeää komennon `systemctl stop`.**Miten PTP saa porttinsa.** PTP käyttää UDP-portteja 319 ja 320, jotka molemmat ovat alle normaalin 1024:n etuoikeutettujen porttien rajan. Paketin `postinst` kirjoittaa `/etc/sysctl.d/60-chloros-ptp.conf`:n yhdessä `net.ipv4.ip_unprivileged_port_start = 319`:n kanssa, mikä antaa taustapalvelimelle mahdollisuuden sitoa ne käyttäjänäsi toimiessaan. Versiosta 1.3.0 lähtien paketti ei aseta tiedosto-oikeuksia (`setcap`) taustaprosessille, eikä `libcap2-bin` ole enää riippuvuus: puratussa taustaprosessissa tiedosto-oikeus estäisi sen käynnistymisen, joten älä lisää sellaista itse.

***

## Bash-skriptiesimerkkejä

{% hint style="info" %}
**Skriptausystävälliset lopetuskoodit.**`chloros-cli process` lopettaa `0`:n onnistuessaan ja**antaa nollasta poikkeavan arvon epäonnistuessaan — mukaan lukien suoritus, joka pyysi kuvatuotteita mutta ei kirjoittanut yhtään** (se tulostaa `Processing finished but wrote no image products.` ja mainitsee projektikansion sekä tavanomaiset syyt). Onnistuneet suoritukset ilmoittavat, kuinka monta kuvatuotetta tallennettiin (`Image products written: N`, sekä `(plus M clamp masks)`, kun maskeja tallennettiin). Lopetuskoodit: `0` onnistuminen, `1` epäonnistuminen, `2` argumenttivirhe, `130` keskeytys.
{% endhint %}

### Useiden tietojoukkojen käsittely

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    if chloros-cli process "$dataset" --format "TIFF (32-bit, Percent)"; then
        echo "Done: $(basename "$dataset")"
    else
        echo "FAILED: $(basename "$dataset")" >&2
    fi
done
```

### Käsittely mukautetuilla asetuksilla

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI GNDVI OSAVI \
    --debayer texture-aware \
    --no-vignette
```

Kelvollisia `--format`-arvoja on tarkalleen neljä, ja ne sisältävät välilyöntejä — laita ne aina lainausmerkkien sisään:

| `--format`-arvo | Tulostuskansio |
| --- | --- |
| `TIFF (16-bit)` *(oletus)* | `tiff16` |
| `TIFF (8-bit)` | `tiff8` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` hyväksyy `standard` (oletus) tai `texture-aware` (Chloros+).

### Automaattinen käsittely Cronin avulla

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Python-SDK-esimerkki

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "OSAVI"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Vianmääritys

### CLI:tä ei löydy asennuksen jälkeen

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### Käyttöoikeus evätty

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend /usr/lib/chloros/backend/chloros-backend
```

### Taustapalvelu ei käynnisty tiedosto-oikeuksien lisäämisen jälkeen

Älä suorita komentoa `setcap` taustapalvelussa. Versiosta 1.3.0 lähtien paketti ei aseta tiedosto-ominaisuuksia, ja PTP:n portit 319/320 tulevat `sysctl`-asetuksesta, joka on kuvattu kohdassa [Kuinka PTP saa porttinsa](#always-on-ptp-for-headless-hosts). Tiedosto-ominaisuus asettaa purkatun taustaprosessin suojattuun suoritusmoodiin, jossa se ei löydä omia kirjastojaan, joten se ei käynnisty. Jos olet lisännyt sellaisen, poista se:

```bash
sudo setcap -r /usr/lib/chloros/backend/chloros-backend
```

### PTP ei käynnisty / ei voi sitoa porttia 319

Varmista, että rajoitetun portin alaraja on laskettu, ja sovella sitä uudelleen nykyiseen käynnistykseen, jos sitä ei ole vielä tehty:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Tarkista sitten grandmaster:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### `lattice`-komennot epäonnistuvat virheilmoituksella ”LATTICE-kameran ajureita ei löydy”

Backend ei pystynyt lataamaan sen mukana asennettua LATTICE-kameran ajoympäristöä. Jokainen `lattice`-komento suoritetaan backendissä; CLI-käyttöliittymä itsessään ei sisällä kameran ajureita. Asenna Chloros uudelleen (`sudo apt install --reinstall ./chloros_1.3.0_<arch>.deb`): `.deb` asentaa ajoympäristön taustapalvelimen viereen, `/usr/lib/chloros/arena_runtime`, ja rekisteröi sen dynaamiseen lataajaan. Voit tarkistaa rekisteröinnin seuraavasti:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
ls /usr/lib/chloros/arena_runtime | head
```

### Taustapalvelimen käynnistys epäonnistui

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

Epäonnistuneen käynnistyksen taustapalvelun lokit ovat tiedostossa `~/.cache/chloros/logs/`. Jos portti 5000 kuuluu taustapalvelulle (`systemctl status chloros-backend@$USER.service`), käynnistä palvelu uudelleen sen sijaan, että lopettaisit sen prosessin.

### CUDA:ta ei havaittu

```bash
# Check NVIDIA driver installation (x86_64)
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check the L4T release (JetPack 6 is R36) and that JetPack's CUDA libraries are installed
cat /etc/nv_tegra_release
sudo apt install nvidia-jetpack
```

`chloros-cli selftest` raportoi saman asian yhdellä rivillä: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Puuttuvat jaetut kirjastot

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries (the program itself, not the launcher script)
ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"
```

### Hidas käynnistys SD-korttijärjestelmissä

Versiosta 1.3.0 lähtien taustapalvelu ja CLI-sovellus asennetaan pakkaamattomina, joten ne eivät enää pura itseään väliaikaiseen hakemistoon jokaisen käynnistyksen yhteydessä. Väliaikaiset tiedostot tallennetaan automaattisesti kansioon `/mnt/ssd/tmp`, jos kyseinen kansio on olemassa; muussa tapauksessa voit ohjata ne nopeaan tiedostojärjestelmään:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Chlorosin päivittäminen Linuxissa

`update`-komento on käytettävissä vain Linuxissa/Jetsonissa. Se tarkistaa `/etc/chloros/update.conf`-asetuksessa määritetyssä päivityskanavassa julkaistun version ja tarjoaa vastaavan `.deb`-päivityksen lataamista ja asentamista:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

Linux- ja Jetson-alustoilla CLI suorittaa myös jokaisella käynnistyksellä ei-estävän päivitystarkistuksen (tulos tallennetaan välimuistiin tunniksi kohdassa `~/.chloros/update_cache.json`) ja tulostaa viestin `Update available: vX.Y.Z`, jos uudempi versio on saatavilla. Asetuksesi, projektisi ja kirjautumistietosi säilyvät päivityksen jälkeen.

Paketin päivittäminen pysäyttää kaikki taustapalvelut asennuksen ajaksi, joten käynnissä oleva skripti menettää kameransa ja anturinsa ja käynnistää sitten uudelleen kaikki käytössä olevat (käsin käynnistetty mutta koskaan käyttöön otettu palvelu pysyy pysäytettynä, kuten uudelleenkäynnistyksen jälkeen).

## Poistaminen

```bash
sudo apt remove chloros
```

Poisto pysäyttää ja poistaa käytöstä kaikki taustapalvelut, palauttaa oletusarvoisen rajoitetun porttialueen (1024), poistaa vastaanottopuskuri-asetustiedoston (käynnissä olevat arvot säilyvät uudelleenkäynnistykseen asti), mukana toimitetun exiftool-symbolisen linkin, Arena-lataimen määritykset, paketin lisäämät palomuurisäännöt sekä sen asentaman Python-SDK:n ja tyhjentää välimuistissa olevat tunnistetiedot. Projektisi ja `~/.chloros/`-datatiedostosi säilyvät ennallaan. `sudo apt purge chloros` poistaa myös `/var/log/chloros`:n.

***

## Seuraavat vaiheet

* [NVIDIA Jetson -opas](nvidia-jetson-guide.md) — Jetson-laitteille ominaiset optimoinnit ja käyttöönotto
* [CLI: komentorivi](../CLI.md) — CLI-opas
* [API: Python SDK](../api-python-sdk.md) — SDK-opas
* [CLI-viite](../reference/cli-reference.md) ja [SDK-viite](../reference/sdk-reference.md) — kattavat komento- ja API-luettelot versioon 1.3.0
* [Dynaaminen laskentakapasiteetin mukautus](../processing-architecture/dynamic-compute-adaptation.md) — miten Chloross mukautuu laitteistoosi
