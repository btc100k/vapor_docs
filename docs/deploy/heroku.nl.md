# Wat is Heroku

Heroku is een populaire alles in één hosting oplossing, u kunt meer vinden op [heroku.com](https://www.heroku.com)

## Aanmelden

U heeft een heroku account nodig, als u die niet heeft, kunt u zich hier aanmelden: [https://signup.heroku.com/](https://signup.heroku.com/)

## Installeren CLI

Zorg ervoor dat je de heroku cli tool hebt geïnstalleerd.

### HomeBrew

```bash
brew install heroku/brew/heroku
```

### Andere Installatiemogelijkheden

Zie alternatieve installatieopties hier: [https://devcenter.heroku.com/articles/heroku-cli#download-and-install](https://devcenter.heroku.com/articles/heroku-cli#download-and-install).

### Inloggen

Zodra je de cli geïnstalleerd hebt, log je in met het volgende:

```bash
heroku login
```

controleer of het juiste emailadres is ingelogd met:

```bash
heroku auth:whoami
```

### Maak een applicatie aan

Ga naar dashboard.heroku.com om toegang te krijgen tot uw account, en maak een nieuwe applicatie aan via de dropdown in de rechterbovenhoek. Heroku zal een paar vragen stellen zoals de regio en de naam van de applicatie, volg gewoon hun aanwijzingen.

### Git

Heroku gebruikt Git om je app te implementeren, dus je zult je project in een Git repository moeten zetten, als dat nog niet het geval is.

#### Initialiseer Git

Als je Git aan je project moet toevoegen, voer dan het volgende commando in Terminal in:

```bash
git init
```

#### Master/Main

Standaard deponeert Heroku de **master/main** branch. Zorg ervoor dat alle wijzigingen in deze branch zijn gecontroleerd voordat u gaat pushen.

Controleer uw huidige branch met

```bash
git branch
```

De asterisk geeft de huidige branch aan.

```bash
* master
  commander
  other-branches
```

!!! note "Opmerking"
    Als je geen uitvoer ziet en je hebt net `git init` uitgevoerd. Je moet eerst je code committen, daarna krijg je uitvoer te zien van het `git branch` commando.

Als u momenteel _niet_ op **master/main** bent, schakel daar dan over door in te voeren:

```bash
git checkout master
```

#### Veranderingen Vastleggen

Als dit commando uitvoer produceert, dan heb je nog niet vastgelegde wijzigingen.

```bash
git status --porcelain
```

Leg ze vast met het volgende commando

```bash
git add .
git commit -m "a description of the changes I made"
```

#### Maak verbinding met Heroku

Verbind uw app met heroku (vervang door de naam van uw app).

```bash
$ heroku git:remote -a your-apps-name-here
```

### Set Buildpack

Stel het buildpack in om heroku te leren hoe om te gaan met vapor.

```bash
heroku buildpacks:set vapor/vapor
```

### Swift versie bestand

Het buildpack dat we hebben toegevoegd zoekt naar een **.swift-version** bestand om te weten welke versie van swift gebruikt moet worden. (vervang 5.2.1 door de versie die uw project nodig heeft).

```bash
echo "5.2.1" > .swift-version
```

Dit creëert **.swift-version** met `5.2.1` als inhoud.


### Procfile

Heroku gebruikt het **Procfile** om te weten hoe uw app moet draaien, in ons geval moet het er als volgt uitzien:

```
web: Run serve --env production --hostname 0.0.0.0 --port $PORT
```

we kunnen dit maken met het volgende terminal commando

```bash
echo "web: Run serve --env production" \
  "--hostname 0.0.0.0 --port \$PORT" > Procfile
```

### Veranderingen Vastleggen

We hebben net deze bestanden toegevoegd, maar ze zijn niet gecommit. Als we pushen, zal heroku ze niet vinden.

Leg ze vast met het volgende.

```bash
git add .
git commit -m "adding heroku build files"
```

### Deployen naar Heroku

Je bent klaar om uit te rollen, voer dit uit vanaf de terminal. Het kan een tijdje duren om te bouwen, dit is normaal.

```none
git push heroku master
```

### Scale Up

Als je eenmaal succesvol hebt gebouwd, moet je ten minste één server toevoegen, één web is gratis en je kunt het krijgen met het volgende:

```bash
heroku ps:scale web=1
```

### Continued Deployment

Elke keer dat je wil updaten, zet je gewoon de laatste veranderingen in master en push je naar heroku en het zal opnieuw deployen

## Postgres

### PostgreSQL Database Toevoegen

Bezoek uw applicatie op dashboard.heroku.com en ga naar de **Add-ons** sectie.

Voer hier `postgress` in en u zult een optie zien voor `Heroku Postgres`. Selecteer deze.

Kies het hobby dev free plan, en provision. Heroku zal de rest doen.

Zodra je klaar bent, zie je de database verschijnen onder de **Resources** tab.

### Configureer De Database

We moeten onze app nu vertellen hoe hij toegang krijgt tot de database. In onze app directory, laten we uitvoeren.

```bash
heroku config
```

Dit zal uitvoer geven zoals dit

```none
=== today-i-learned-vapor Config Vars
DATABASE_URL: postgres://cybntsgadydqzm:2d9dc7f6d964f4750da1518ad71hag2ba729cd4527d4a18c70e024b11cfa8f4b@ec2-54-221-192-231.compute-1.amazonaws.com:5432/dfr89mvoo550b4
```

**DATABASE_URL** hier zal de postgres database staan. Hard code de statische url **NOOIT**, heroku zal het roteren en het zal uw toepassing breken. Het is ook een slechte praktijk. Lees in plaats daarvan de omgevingsvariabele tijdens runtime.

De Heroku Postgres addon [vereist](https://devcenter.heroku.com/changelog-items/2035) dat alle verbindingen versleuteld zijn. De certificaten die door de Postgres servers worden gebruikt zijn intern aan Heroku, daarom moet een **ongeverifieerde** TLS verbinding worden opgezet.

Het volgende fragment laat zien hoe beide bereikt kunnen worden:

```swift
if let databaseURL = Environment.get("DATABASE_URL"), var postgresConfig = PostgresConfiguration(url: databaseURL) {
    postgresConfig.tlsConfiguration = .makeClientConfiguration()
    postgresConfig.tlsConfiguration?.certificateVerification = .none
    app.databases.use(.postgres(configuration: postgresConfig), as: .psql)
} else {
    // ...
}
```

Vergeet niet om deze wijzigingen vast te leggen

```none
git add .
git commit -m "configured heroku database"
```

### Uw Database Terugdraaien

U kunt andere commando's op heroku terugdraaien of uitvoeren met het `run` commando. Het project van Vapor heet standaard ook `Run`, dus het leest een beetje raar.

Om uw database terug te zetten:

```bash
heroku run Run -- migrate --revert --all --yes --env production
```

Om te migreren

```bash
heroku run Run -- migrate --env production
```