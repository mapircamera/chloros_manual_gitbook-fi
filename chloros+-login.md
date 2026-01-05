# Chloros+ Kirjautuminen

## Chloros ja Chloros (selain) Kirjautuminen

Käyttäjän <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> sivupalkin valikosta voit kirjautua Chloros+ -tiliisi ja avata lisäominaisuuksia.

Kun olet kirjautunut sisään, tilisi tiedot näkyvät:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" data-size="line"><figcaption></figcaption></figure>## CLI Kirjautuminen

Kirjaudu sisään Chloros+ -tunnuksillasi, jotta CLI-käsittely voidaan ottaa käyttöön.

**Syntaksi:**

```bash
chloros-cli login <email> <password>
```

{% hint style=&quot;info&quot; %}
**SDK-käyttäjät**: Python SDK tarjoaa myös ohjelmoitavan `logout()`-menetelmän välimuistissa olevien tunnistetietojen tyhjentämiseen. Katso lisätietoja [Python SDK-dokumentaatiosta](api-python-sdk.md#logout).
{% endhint %}

**Esimerkki:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**Erikoismerkit**: Käytä yksittäisiä lainausmerkkejä salasanojen ympärillä, jotka sisältävät merkkejä kuten `$`, `!` tai välilyöntejä.
{% endhint %}

**Tulos:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### Suunnitelman voimassaolon päättyminen

GUI:ssa näkyvä suunnitelman voimassaolon päättyminen osoittaa, milloin lisenssisi vanhenee. Kuukausittaisten toistuvien tilausten voimassaolo päättyy kuukauden lopussa. Vuosittaisten tilausten voimassaolo päättyy vuoden kuluttua tilauksen aloittamisesta. Lisenssin tarkistaminen edellyttää kuukausittaista internetyhteyttä, ja sen tarkistamiseen on 30 päivän lisäaika.

### Laiterajoitus

Jokainen Chloros+ -sopimus tarjoaa eri määrän rekisteröityjä laitteita. Jokainen laite, johon kirjaudut Chloros+ -tilillä, lasketaan rekisteröityjen laitteiden määrään. Voit nimetä laitteen uudelleen ja poistaa sen MAPIR Cloud -tilisi sivulta.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+ -sopimus</th><th align="center">KUPARI</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">KULTA</th></tr></thead><tbody><tr><td align="right">Tuetut laitteet</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
