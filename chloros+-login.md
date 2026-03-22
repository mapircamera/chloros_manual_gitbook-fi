# Chloros+ -kirjautuminen

## Chloros ja Chloros (selain) -kirjautuminen

Käyttäjän <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> sivupalkin valikosta voit kirjautua Chloros+ -tilillesi ja avata lisäominaisuuksia.

Kun olet kirjautunut sisään, tilitietosi näkyvät:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>## CLI-kirjautuminen

Kirjaudu sisään Chloros+ -tunnuksillasi, jotta voit ottaa CLI-käsittelyn käyttöön. Linux:ssä (ei graafista käyttöliittymää) tämä on ainoa tapa aktivoida lisenssisi.

**Syntaksi:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**SDK-käyttäjät**: Python SDK tarjoaa myös ohjelmointitason `logout()`-menetelmän välimuistissa olevien tunnistetietojen tyhjentämiseen. Katso lisätietoja [Python SDK-dokumentaatiosta](api-python-sdk.md#logout).
{% endhint %}

**Esimerkki:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Erikoismerkit**: Käytä yksinkertaisia lainausmerkkejä salasanojen ympärillä, jotka sisältävät merkkejä kuten `$`, `!` tai välilyöntejä.
{% endhint %}

**Tulostus:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### Tunnistetietojen tallennus

Välimuistiin tallennetut tunnistetiedot säilytetään alustakohtaisessa sijainnissa:

| Alusta | Tunnistetietojen välimuistin polku |
| --- | --- |
| **Windows** | `%APPDATA%\Chloros\cache\` |
| **Linux** | `~/.cache/chloros/` |

### Paketin voimassaolon päättyminen

GUI:ssa näkyvä paketin voimassaolon päättyminen osoittaa, milloin lisenssisi vanhenee. Toistuvien kuukausitilauksien voimassaolo päättyy kuukauden lopussa. Vuositilauksissa voimassaolo päättyy vuoden kuluttua tilauksen aloittamisesta. Lisenssin tarkistus vaatii kuukausittaisen internetyhteyden vahvistusta varten, ja siihen sisältyy 30 päivän armonaika.

### Laiterajoitus

Jokainen Chloros+ -sopimus tarjoaa eri määrän rekisteröityjä laitteita. Jokainen laite, johon kirjaudut Chloros+ -tilillä, lasketaan mukaan rekisteröityjen laitteiden määrään. Voit nimetä laitteen uudelleen ja poistaa sen MAPIR Cloud -tilisivultasi.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+ -paketti</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">Tuetut laitteet</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
