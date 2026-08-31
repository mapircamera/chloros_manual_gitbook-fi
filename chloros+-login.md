# Chloros+ Kirjautuminen

## Kirjautuminen käyttöliittymän kautta

Käyttäjävalik<img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">in sivupalkissa voit kirjautua Chloros+-tilillesi ja avata lisäominaisuuksia.

**Sinun tarvitsee kirjautua sisään vain kerran kutakin laitetta kohti.** Käyttöliittymä, CLI, ja Python sekä SDK jakavat saman välimuistissa olevan istunnon — kirjautuminen työpöydän käyttöliittymän kautta aktivoi myös CLI- ja SDK-sovellukset kyseisellä laitteella (ja päinvastoin `chloros-cli login`:n kautta).

Kun olet kirjautunut sisään, tilitietosi näytetään:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.2.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->
## Pakettitasot

| Paketti | `plan_id` | Tyyppi |
| --- | --- | --- |
| Iron | `0` | Ilmainen |
| Copper | `1` | Maksullinen (Chloros+) |
| Bronze | `2` | Maksullinen (Chloros+) |
| Hopea | `3` | Maksullinen (Chloros+) |
| Kulta | `4` | Maksullinen (Chloros+) |

Katso [paketit ja hinnat](https://cloud.mapir.camera/pricing) saadaksesi tietää, mitä kukin maksullinen taso sisältää.

### CLI / SDK -käyttöoikeus edellyttää maksullista tasoa

CLI- ja Python- sekä SDK-käyttöoikeudet edellyttävät **mitä tahansa maksullista Chloros+-tasoa (Copper tai korkeampi)**. Tämä valvotaan**palvelinpuolella** — jokaisessa CLI/SDK-pyynnössä on oltava sekä aktiivinen istunto että maksullinen tilaus:

| HTTP-tila | `error_code` | Merkitys | Korjaus |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | Ei kirjautunut sisään tällä laitteella | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | Kirjautunut sisään, mutta palvelutason taso on liian alhainen (ilmainen Iron-taso) | Päivitä mihin tahansa maksulliseen Chloros+-palvelutasoon |

`chloros-cli status` on edelleen käytettävissä ilmaisella tasolla, joten voit aina tarkistaa nykyisen tilauksesi ja syyn, miksi pääsy evättiin.

### Kytkettyjen laitteiden rajoitukset tilauskohtaisesti

Jokaisella tilauksella on yläraja sille, kuinka monta LATTICE-kameraa ja DAQ-valosensoria voidaan kytkeä reaaliaikaisesti kerrallaan:

| Paketti | LATTICE-kamerat | DAQ-valosensorit |
| --- | --- | --- |
| Iron (ilmainen / kirjautumaton) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## CLI-kirjautuminen

Kirjaudu sisään Chloros+-tunnuksillasi, jotta voit ottaa CLI-käsittelyn käyttöön. Linux-versiossa (ilman graafista käyttöliittymää) tämä on ainoa tapa aktivoida lisenssisi.

**Syntaksi:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**SDK-käyttäjät**: Python SDK tarjoaa myös ohjelmointitason `logout()`-menetelmän välimuistissa olevien tunnistetietojen tyhjentämiseen. Katso lisätietoja [SDK-viitteestä](reference/sdk-reference.md).
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
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.2.0"; capture a successful login with the current output format. -->
### Tunnistetietojen tallennus

Välimuistiin tallennetut tunnistetiedot ja asetukset tallennetaan käyttäjän kotihakemistossa olevaan `.chloros`-kansioon **kaikilla alustoilla**:

| Alusta | Tunnistetietojen välimuistin polku |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

### Tilauksen voimassaolon päättyminen ja siirtymäaika

Käyttöliittymässä näkyvä tilauksen voimassaolon päättymispäivä osoittaa, milloin lisenssisi menettää voimassaolonsa. Toistuvien kuukausitilauksien osalta voimassaolo päättyy kuukauden lopussa; vuositilauksissa voimassaolo päättyy vuoden kuluttua tilauksen aloittamisesta.

Chloros tarkistaa lisenssisi verkossa, mutta offline-käyttö on mahdollista siirtymäajan puitteissa:

* Onnistuneet palvelintarkistukset tallennetaan välimuistiin **5 minuutiksi**, joten normaalikäytössä lisenssipyyntöjä syntyy hyvin vähän.
* Allekirjoitettu, laitteeseen sidottu lisenssivälimuisti kattaa pidemmät offline-jaksot: **30 päivää kuukausitilauksissa**ja**vuosittaisissa tilauksissa tilauksen voimassaolon päättymispäivään asti (enintään 365 päivää)**.
* Kun armonaika päättyy, tilaus siirtyy ilmaiseen Iron-tasoon, kunnes laite pystyy muodostamaan yhteyden lisenssipalvelimeen kerran; käyttöoikeus palautuu seuraavan onnistuneen tarkistuksen yhteydessä.

### Laiterajoitus

Jokainen Chloros+-tilaus tarjoaa eri määrän rekisteröityjä laitteita. Jokainen laite, jolla kirjaudut sisään Chloros+-tilillä, lasketaan mukaan rekisteröityjen laitteiden määrään. Voit nimetä laitteen uudelleen ja poistaa sen MAPIR Cloud -tilisivultasi.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+-paketti</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">Tuetut laitteet</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>Tilin tarkka laitemäärä näkyy MAPIR Cloud -tilisivulla. Laitteesta kirjautuminen vapauttaa sen paikan luotettavasti, ja jo rekisteröity laite voi aina kirjautua takaisin sisään, vaikka tilin laitemääräraja olisi täynnä.
