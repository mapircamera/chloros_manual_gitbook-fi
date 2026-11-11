# Chloros+ -kirjautuminen

## Kirjautuminen graafisen käyttöliittymän kautta

Käyttäjän sivupalkin valikosta <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> voit kirjautua Chloros+ -tiliisi ja avata lisäominaisuuksia.

**Sinun tarvitsee kirjautua sisään vain kerran kutakin laitetta kohden.** GUI, CLI ja Python-SDK käyttävät samaa välimuistissa olevaa istuntoa — kirjautuminen työpöydän GUI:n kautta aktivoi myös CLI:n ja SDK:n kyseisellä laitteella (ja päinvastoin `chloros-cli login`:n kautta).

Kun olet kirjautunut sisään, tilitietosi näytetään:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.3.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->

## Tilauspaketit

| Tilauspaketti | `plan_id` | Tyyppi |
| --- | --- | --- |
| Iron | `0` | Ilmainen |
| Copper | `1` | Maksullinen (Chloros

+) |
| Bronze | `2` | Maksullinen (Chloros

+) |
| Silver | `3` | Maksullinen (Chloros

+) |
| Kulta | `4` | Maksullinen (Chloros

+) |

Katso [paketit ja hinnat](https://cloud.mapir.camera/pricing) saadaksesi tietää, mitä kukin maksullinen taso sisältää.

### CLI

- ja SDK

-käyttöoikeus edellyttää maksullista tasoa

CLI

:n ja Python

-SDK

:n käyttö edellyttää **mitä tahansa maksullista Chloros

+-tasoa (Copper tai korkeampi)**. Tämä valvotaan**palvelinpuolella** — jokaisessa CLI

/SDK

-pyynnössä on oltava sekä aktiivinen istunto että maksullinen paketti:

| HTTP

-tila | `error_code` | Merkitys | Korjaus |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | Ei kirjautunut sisään tällä laitteella | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | Kirjautunut sisään, mutta tilaus on liian alhainen (ilmainen Iron-taso) | Päivitä mihin tahansa maksulliseen Chloros

+-tilaukseen |

`chloros-cli status` on edelleen käytettävissä ilmaisella tasolla, joten voit aina tarkistaa nykyisen tilauksesi ja syyn, miksi pääsy evättiin.

### Liitettyjen laitteiden rajoitukset tilauskohtaisesti

Jokaisella tilauksella on yläraja sille, kuinka monta LATTICE-kameraa ja DAQ-valosensoria voidaan liittää reaaliaikaisesti kerrallaan:

| Taso | LATTICE-kamerat | DAQ-valosensorit |
| --- | --- | --- |
| Iron (ilmainen / kirjautumaton) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## CLI

-kirjautuminen

Kirjaudu sisään Chloros

+-tunnuksillasi CLI

-käsittelyn ottamiseksi käyttöön. Linux

issa (ilman graafista käyttöliittymää) tämä on ainoa tapa aktivoida lisenssisi.

**Syntaksi:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**SDK

-käyttäjät**: Python

-SDK

tarjoaa myös ohjelmointimenetelmän `logout()` välimuistissa olevien tunnistetietojen tyhjentämiseen. Katso lisätietoja [SDK

-viitteestä](reference/sdk-reference.md).
{% endhint %}

**Esimerkki:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Erikoismerkit**: Käytä yksinkertaisia lainausmerkkejä salasanojen ympärillä, jos ne sisältävät merkkejä kuten `$`, `!` tai välilyöntejä.
{% endhint %}

**Tulostus:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.3.0"; capture a successful login with the current output format. -->

### Tunnistetietojen tallennus

Välimuistiin tallennetut tunnistetiedot ja asetukset tallennetaan käyttäjän kotihakemistossa olevaan `.chloros`-kansioon **kaikilla alustoilla**:

| Alusta | Tunnistetietojen välimuistin polku |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

Uudemman version asentaminen olemassa olevan version päälle säilyttää kirjautumistietosi Windows

issa (asennusohjelma) ja Linux

issa (`.deb`, mukaan lukien `chloros-cli update`). Chloros

in poistaminen — tai Linux

-paketin poistaminen — tyhjentää välimuistissa olevan kirjautumistiedon, joten sinun on kirjauduttava uudelleen sisään uuden asennuksen jälkeen.

### Tilauksen voimassaolon päättyminen ja offline-armonjakso

GUI:ssa näkyvä tilauksen voimassaolon päättymispäivä osoittaa, milloin lisenssisi menettää voimassaolonsa. Toistuvien kuukausitilauksien voimassaolo päättyy kuukauden lopussa; vuositilauksissa voimassaolo päättyy vuoden kuluttua tilauksen aloittamisesta.

Chloros

tarkistaa lisenssisi verkossa, mutta offline-käyttö on tuettu siirtymäajan puitteissa:

* Onnistuneet palvelintarkistukset tallennetaan välimuistiin **5 minuutiksi**, joten normaalikäytössä lisenssikyselyjä tarvitaan hyvin vähän.
* Allekirjoitettu, laitteeseen sidottu lisenssivälimuisti kattaa pidemmät offline-jaksot: **30 päivää kuukausitilauksissa**ja**vuosittaisissa tilauksissa tilauksen voimassaolon päättymispäivään asti (enintään 365 päivää)**.
* Kun siirtymäaika päättyy, tilaus siirtyy ilmaiseen Iron-tasoon, kunnes laite pääsee yhteyteen lisenssipalvelimen kanssa; käyttöoikeus palautuu seuraavan onnistuneen tarkistuksen yhteydessä.

### Laiterajoitus

Jokainen Chloros

+-sopimus tarjoaa eri määrän rekisteröityjä laitteita. Jokainen laite, johon kirjaudut Chloros

+-tililläsi, lasketaan mukaan rekisteröityjen laitteiden määrään. Voit nimetä laitteen uudelleen ja poistaa sen MAPIR

-pilvipalvelun tilisivulta.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros-tilaus</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">Tuetut laitteet</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>

Tilin tarkka laitemäärä näkyy MAPIR-pilvipalvelun tilisivulla. Laitteesta kirjautuminen vapauttaa sen paikan varmasti, ja jo rekisteröity laite voi aina kirjautua takaisin sisään, vaikka tilin laitemääräraja olisi täynnä.
