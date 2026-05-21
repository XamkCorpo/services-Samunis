# Vaihe 3: Service-kerros, Repository, Result Pattern ja API-dokumentaatio — Teoriakysymykset

Vastaa alla oleviin kysymyksiin omin sanoin. Kirjoita vastauksesi kysymysten alle.

> **Vinkki:** Jos jokin kysymys tuntuu vaikealta, palaa lukemaan teoriamateriaalit:
> - [Service-kerros ja DI](https://github.com/xamk-mire/Xamk-wiki/blob/main/C%23/fin/04-Advanced/WebAPI/Services-and-DI.md)
> - [Repository Pattern](https://github.com/xamk-mire/Xamk-wiki/blob/main/C%23/fin/04-Advanced/Patterns/Repository-Pattern.md)
> - [Result Pattern](https://github.com/xamk-mire/Xamk-wiki/blob/main/C%23/fin/04-Advanced/Patterns/Result-Pattern.md)

---

## Osa 1: Service-kerros

### Kysymys 1: Fat Controller -ongelma

Miksi on ongelma jos controller sisältää kaiken logiikan (tietokantakyselyt, muunnokset, validoinnin)? Anna vähintään kaksi konkreettista haittaa.

Jos controller sisältää kaiken logiikan, se johtaa "fat controller" -ongelmaan, joka tarkoittaa, että controllerista tulee liian monimutkainen ja vaikeasti ylläpidettävä. Tässä haitat:
1. Testattavuus heikkenee, koska yksittäisen metodin testaaminen vaatii koko controllerin kontekstin.
2. Koodin uudelleenkäytettävyys vähenee, koska logiikka on sidottu tiettyyn controlleriin eikä sitä voi helposti käyttää muissa osissa sovellusta.

---

### Kysymys 2: Vastuunjako

Miten vastuut jakautuvat controller:n, service:n ja repository:n välillä tässä harjoituksessa? Kirjoita lyhyt kuvaus kunkin kerroksen tehtävästä.

**Controller vastaa:** HTTP-pyyntöjen vastaanottamisesta.

**Service vastaa:** Logiikasta ja DTO-muunnoksista.

**Repository vastaa:** Tietokantakyselyistä ja entiteettien käsittelystä.


---

### Kysymys 3: DTO-muunnokset servicessä

Miksi DTO ↔ Entity -muunnokset kuuluvat serviceen eikä controlleriin? Mitä hyötyä siitä on, että controller ei tunne `Product`-entiteettiä lainkaan?

Muunnokset kuuluvat serviceen, koska se on vastuussa logiikasta ja datan käsittelystä. Controllerin tehtävä on vain vastaanottaa HTTP-pyyntö ja palauttaa HTTP-vastaus, joten sen ei tarvitse tietää tietokantamalleista. Tämä erottelu parantaa koodin selkeyttä, testattavuutta ja ylläpidettävyyttä, koska controller ei ole sidottu tiettyyn tietomalliin ja service voi muuttaa entiteettejä ilman, että controller tarvitsee muuttua.


---

## Osa 2: Interface ja Dependency Injection

### Kysymys 4: Interface vs. konkreettinen luokka

Miksi controller injektoi `IProductService`-interfacen eikä suoraan `ProductService`-luokkaa? Mitä hyötyä tästä on?

koska se mahdollistaa löyhemmän kytkennän ja helpottaa testattavuutta.


---

### Kysymys 5: DI-elinkaaret

Selitä ero näiden kolmen elinkaaren välillä ja anna esimerkki milloin kutakin käytetään: 

- **AddScoped:** Luodaan uusi instanssi jokaista HTTP-pyyntöä kohden. Käytetään esimerkiksi palveluille, jotka tarvitsevat tietokantayhteyden per pyyntö.
- **AddSingleton:** Luodaan yksi instanssi koko sovelluksen elinkaaren ajaksi. Käytetään esimerkiksi konfiguraatio- tai cache-palveluille.
- **AddTransient:** Luodaan uusi instanssi jokaista riippuvuutta kohden. Käytetään esimerkiksi lyhytaikaisille, tilapäisille palveluille.

Miksi `AddScoped` on oikea valinta `ProductService`:lle?
ProductService tarvitsee tietokantayhteyden, joka on yleensä määritetty scoped-elinkaareksi.


---

### Kysymys 6: DI-kontti

Selitä omin sanoin mitä DI-kontti tekee kun HTTP-pyyntö saapuu ja `ProductsController` tarvitsee `IProductService`:ä. Mitä tapahtuu vaihe vaiheelta?

DI-kontti huomaa, että ProductsController tarvitsee IProductService:ä, luo automaattisesti ProductService-olion sen riippuvuuksineen ja antaa sen controllerille.


---

### Kysymys 7: Rekisteröinnin unohtaminen

Mitä tapahtuu jos unohdat rekisteröidä `IProductService`:n `Program.cs`:ssä? Milloin virhe ilmenee ja miltä se näyttää?

Silloin sovellus ei pysty luomaan ProductService-oliota, ja HTTP-pyyntöihin vastataan 500 Internal Server Error -virheellä, jossa lukee jotain tyyliin "Unable to resolve service for type IProductService while attempting to activate ProductsController."


---

## Osa 3: Repository-kerros

### Kysymys 8: Miksi repository?

`ProductService` käytti aluksi `AppDbContext`:ia suoraan. Miksi se refaktoroitiin käyttämään `IProductRepository`:a? Anna vähintään kaksi syytä.

koska se parantaa koodin erottelua, testattavuutta ja ylläpidettävyyttä. Repository-kerros kapseloi tietokantakyselyt, joten service ei tarvitse tietää tietokannan rakenteesta tai ORM:stä, mikä tekee siitä joustavamman ja helpommin testattavan.


---

### Kysymys 9: Service vs. Repository

Mikä on `IProductService`:n ja `IProductRepository`:n välinen ero? Mitä tietotyyppejä kumpikin käsittelee (DTO vai Entity)?

**IProductService:** käsittelee DTO ja sisältää liiketoimintalogiikkaa, validointia ja muunnoksia.

**IProductRepository:** käsittelee Entity ja sisältää vain tietokantakyselyt ja operaatiot.


---

### Kysymys 10: Controllerin muuttumattomuus

Kun Vaihe 7:ssä lisättiin repository-kerros, `ProductsController` ei muuttunut lainkaan. Miksi? Mitä tämä kertoo rajapintojen (interface) hyödystä?

Koska controller ei ole sidottu tiettyyn toteutukseen, vaan se käyttää rajapintaa (IProductService), joka pysyy samana. Tämä kertoo, että rajapinnat mahdollistavat joustavuuden ja erottelun, koska ne määrittelevät vain sopimuksen ilman toteutusta, mikä tekee koodista helpommin ylläpidettävää ja testattavaa.


---

## Osa 4: Exception-käsittely ja lokitus

### Kysymys 11: ILogger

Mikä on `ILogger` ja miksi sitä tarvitaan? Mistä lokit näkee kehitysympäristössä?

ILogger on ASP.NET Core -sovelluksissa käytetty rajapinta, joka mahdollistaa lokiviestien kirjoittamisen eri tasoilla. Sitä tarvitaan, jotta sovelluksen tapahtumia ja virheitä voidaan seurata ja diagnosoida. Kehitysympäristössä lokit näkyvät yleensä konsolissa, jossa sovellus ajetaan.


---

### Kysymys 12: Odotetut vs. odottamattomat virheet

Selitä ero "odotetun" ja "odottamattoman" virheen välillä. Anna esimerkki kummastakin ja kerro miten ne käsitellään eri tavalla servicessä.

**Odotettu virhe (esimerkki + käsittely):** Jos käyttäjä yrittää hakea tuotetta, joka ei ole olemassa, se on odotettu virhe. Tällöin service palauttaa Result.Failure-objektin, joka kertoo että tuote ei löytynyt, ja controller käsittelee tämän palauttamalla NotFound-vastauksen.

**Odottamaton virhe (esimerkki + käsittely):** Jos tietokantayhteys epäonnistuu tai tapahtuu muu odottamaton poikkeus, se on odottamaton virhe. Tällöin service voi heittää poikkeuksen, ja controller käsittelee sen esimerkiksi palauttamalla 500 Internal Server Error vastauksen.


---

## Osa 5: Result Pattern

### Kysymys 13: Miksi null ja bool eivät riitä?

Alla on kaksi esimerkkiä. Selitä miksi ensimmäinen tapa on ongelmallinen ja miten toinen ratkaisee ongelman:

```csharp
// Tapa 1: null
ProductResponse? product = await _service.GetByIdAsync(id);
if (product == null)
    return NotFound();

// Tapa 2: Result
Result<ProductResponse> result = await _service.GetByIdAsync(id);
if (result.IsFailure)
    return NotFound(new { error = result.Error });
```

Ensimmäisessä tavassa, jos GetByIdAsync palauttaa null, ei ole selvää miksi tuote ei löytynyt (esim. oliko se todella olemassa vai tapahtuiko jokin muu virhe). Toisessa tavassa Result-objekti sisältää selkeän tilan (IsFailure) ja virheilmoituksen (Error), joka auttaa ymmärtämään, miksi tuote ei löytynyt.


---

### Kysymys 14: Result.Success vs. Result.Failure

Miten `Result Pattern` muutti virheiden käsittelyä servicessä? Vertaa Vaihe 8:n `throw;`-tapaa Vaihe 9:n `Result.Failure`-tapaan: mitä eroa niillä on asiakkaan (API:n kutsuja) näkökulmasta?

Result Pattern tarjoaa selkeämmän ja johdonmukaisemman tavan käsitellä virheitä, koska se ei perustu poikkeuksiin, vaan palauttaa aina onnistuneen tai epäonnistuneen tilan. Asiakkaan näkökulmasta tämä tarkoittaa, että he voivat tarkistaa Result-objektin tilan (IsSuccess/IsFailure) ja saada lisätietoja virheestä ilman, että heidän tarvitsee käsitellä poikkeuksia.


---

## Osa 6: API-dokumentaatio

### Kysymys 15: IActionResult vs. ActionResult\<T\>

Miksi `ActionResult<ProductResponse>` on parempi kuin `IActionResult`? Anna vähintään kaksi syytä.

Koska se tarjoaa paremman tyypintarkistuksen ja dokumentaation, mikä auttaa kehittäjiä ymmärtämään, mitä tyyppiä odotetaan palautettavan. Lisäksi se parantaa Swagger UI:n dokumentaatiota, koska se näyttää selkeästi, että endpoint palauttaa ProductResponse-tyyppisiä vastauksia.


---

### Kysymys 16: ProducesResponseType

Mitä `[ProducesResponseType]`-attribuutti tekee? Miten se näkyy Swagger UI:ssa?

Se kertoo Swagger UI:lle, mitä HTTP-statuskoodeja ja vastaustyyppejä endpoint voi palauttaa. se parantaa API-dokumentaatiota ja auttaa kehittäjiä ymmärtämään, mitä odottaa endpointilta.


---

### Kysymys 18: Refaktorointi

Sovelluksen toiminnallisuus pysyi täysin samana koko harjoituksen ajan — samat endpointit, samat vastaukset. Mitä refaktorointi tarkoittaa ja miksi se kannattaa, vaikka käyttäjä ei huomaa eroa?

Refaktorointi tarkoittaa koodin rakenteen parantamista ilman, että sen ulkoinen käyttäytyminen muuttuu. Se kannattaa, koska se tekee koodista helpommin luettavaa, ylläpidettävää ja testattavaa.


---
