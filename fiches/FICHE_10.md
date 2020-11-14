# Dates, chaînes de caractère et localisation
## Travailler avec les dates et les heures
En Java 8, Oracle à totalement changé la façon de travailler avec les dates et les temps.

Vous aurez besoin d'un `import` pour travailler avec les classes de dates et de temps.

La plupart de ces classes sont dans le package `java.time` :
```java
import java.time.*;
```
> Attention en anglais américain les mots *day* et *date* sont souvent utilisés comme synonymes

### Créer des dates et des heures
- `LocalDate` : contient uniquement une date (pas d'heure, pas de timezone). Exemple : date d'anniversaire, c'est votre anniversaire toute la journée peu importe l'heure,
- `LocalTime` : contient uniquement une heure (pas de date, pas de timezone). Exemple : minuit. Il est minuit tous les jours à la même heure,
- `LocalDateTime` : contient uniquement la date et l'heure (pas de timezone). Exemple : minuit du nouvel an,
- `ZonedDateTime` : contient la date, l'heure et la timezone. Exemple Mardi 11 novembre 03:01 pm CET.

Oracle recommande de n'utiliser les timezones que si nécessaire.

Vous pouvez obtenir une date grace à une méthode statique :
```java
System.out.println(LocalDate.now());        // 2020-11-11
System.out.println(LocalTime.now());        // 14:05:03.105
System.out.println(LocalDateTime.now());    // 2020-11-11T14:05:03.106
System.out.println(ZonedDateTime.now());    // 2020-11-11T14:05:03.107352+01:00[Europe/Paris]
```
Le [temps moyen de Greenwich](https://fr.wikipedia.org/wiki/Temps_moyen_de_Greenwich) est une timezone en europe qui est utilisée comme temps zéro lorsque l'on parle de décalage de fuseaux horaires.

[UTC](https://fr.wikipedia.org/wiki/Temps_universel_coordonn%C3%A9) (Temps universel coordonné) est le nouveau nom du temps moyen de Greenwich.

```
2020-11-11T14:47:24.444123+01:00[Europe/Paris] // GMT 2020-11-11 15h47
2020-11-11T14:47:24.444123+05:30[Asia/Kolkata] // GMT 2020-11-11 20h17
```
En convertissant à l'heure GMT, vous pouvez voir que Paris est 4 heures et demie derriere l'heure de Calcutta.

#### Création d'une `LocalDate`
```java
LocalDate date1 = LocalDate.of(2020, Month.JANUARY, 20);
LocalDate date2 = LocalDate.of(2020, 1, 20)
```
Bien qu'il soit recommandé de passer les constantes de `Month`, on peut passer le numéro du mois directement

Les signatures de la méthode `of` pour `LocalDate` sont :
```java
public static LocalDate of(int year, int month, int dayOfMonth)
public static LocalDate of(int year, Month month, int dayOfMonth)
```

`Month` est un `enum` pas un `int` et ne peut pas être comparée comme tel :
```java
Month month = Month.JANUARY;
boolean b1 = month == 1;            // NE COMPILE PAS
boolean b2 = month == Month.APRIL;  // false 
```
#### Création d'un `LocalTime`
Lorsque vous créez une heure, vous choisissez le niveau de détail que vous souhaitez.
```java
LocalTime time1 = LocalTime.of(6, 15);              // heure et minutes
LocalTime time2 = LocalTime.of(6, 15, 30);          // + secondes   
LocalTime time3 = LocalTime.of(6, 15, 30, 200);     // + nano-secondes   
```
Les signatures de méthode de `of` pour `LocalTime` sont : 
```java
public static LocalTime of(int hour, int minute)
public static LocalTime of(int hour, int minute, int second)
public static LocalTime of(int hour, int minute, int second, int nanos)
```
#### Création d'une `LocalDateTime`
Vous pouvez combiner les dates et heures ensemble :
```java
LocalDateTime dateTime1 = LocalDateTime.of(2015, Month.JANUARY, 20, 6, 5, 30);
LocalDateTime dateTime2 = LocalDateTime.of(date1, time1);
```
Les signatures de méthode de `of` pour `LocalDateTime` sont : 
```java
public static LocalTime of(int year, int month, int dayOfMonth, int hour, int minute)
public static LocalTime of(int year, int month, int dayOfMonth, int hour, int minute, int second)
public static LocalTime of(int year, int month, int dayOfMonth, int hour, int minute, int second, int nanos)

public static LocalTime of(int year, Month month, int dayOfMonth, int hour, int minute)
public static LocalTime of(int year, Month month, int dayOfMonth, int hour, int minute, int second)
public static LocalTime of(int year, Month month, int dayOfMonth, int hour, int minute, int second, int nanos)

public static LocalTime of(LocalDate date, LocalTIme time)
```
#### Création d'une `ZonedDateTime`
```java
ZoneId zone = ZoneId.of("US/Eastern"); 
ZonedDateTime zoned1 = ZonedDateTime.of(2015, 1, 20, 6, 15, 30, 200, zone);
ZonedDateTime zoned2 = ZonedDateTime.of(date1, time1, zone);
ZonedDateTime zoned3 = ZonedDateTime.of(dateTime1, zone);
```

Les signatures de méthode de `of` pour `ZonedDateTime` sont :
```java
public static ZonedDateTime of(int year, int month, int dayOfMonth, int hour, int minute, int nanos, ZoneId zone)
public static ZonedDateTime of(LocalDate date, LocalTime time, ZoneId zone)
public static ZonedDateTime of(LocalDateTime dateTime, ZoneId zone)
``` 

Remarquez qu'on ne peut passer l'`enum` `Month`. C'est une omission des créateurs de l'API qui sera réparée dans des versions futures de Java.
##### Trouver sa timezone
Vous pouvez facilement trouver votre timezone avec `ZoneId.systemDefault()`, ou sinon vous pouvez la rechercher comme ceci :
```java
ZoneID.getAvailableZoneIds().stream()
    .filter(z -> z.contains("US") || z.contains("America"))
    .sorted().forEach(System.out::println);
```
Essayez avec le nom du pays ou le nom de la ville, ou sinon Googlisez la.
##### Pièges
###### Constructeur
Les classes de date et heure vous force à utiliser les *factory methods* via des constructeurs privés.

L'appel au constructeur ne compile pas :
```java
LocalDate d = new LocalDate(); // NE COMPILE PAS
```
###### Nombres invalides
```java
LocalDate.of(2015, Month.JANUARY, 32); // Lance une DateTimeException
```
```
java.time.DateTimeException: Invalid value for DayOfMonth
(valid values 1-28/31): 32
```

#### Manipuler les dates et heures
Les classes de date et heure sont immutables, n'oubliez pas de les assigner à une varible de référence pour ne pas les perdre.
##### Ajouter
```java
LocalDate date = LocalDate.of(2014, Month.JANUARY, 20);
System.out.println(date);   // 2014-01-20

date = date.plusDays(2);
System.out.println(date);   // 2014-01-22

date = date.plusWeeks(1);
System.out.println(date);   // 2014-01-29

date = date.plusMonths(1);
System.out.println(date);   // 2014-02-28 -- java réalise que 2014 n'est pas un année bissextile et nous donne le 28 plutot que le 29

date = date.plusYears(5);
System.out.println(date);   // 2019-02-28
```

> Les années bissextiles sont les de 4 et de 400 mais pas les multiples de 100

##### Soustraire
```java
LocalDate date = LocalDate.of(2020, Month.JANUARY, 20);
LocalTime time = LocalTime.of(5, 15);
LocalDateTime dateTime = LocalDateTime.of(date, time);
System.out.println(dateTime); // 2020-01-20T05:15

dateTime = dateTime.minusDays(1);
System.out.println(dateTime); // 2020-01-19T05:15

dateTime = dateTime.minusHours(10);
System.out.println(dateTime); // 2020-01-18T19:15

dateTime = dateTime.minusSeconds(30);
System.out.println(dateTime); // 2020-01-18T19:14:30
```
Java n'affiche pas les secondes et les nanosecondes lorsque nous ne les utilisons pas.

On peut chaîner les méthodes :
```java
LocalDate date = LocalDate.of(2020, Month.JANUARY, 20);
LocalTime time = LocalTime.of(5, 15);
LocalDateTime dateTime = LocalDateTime.of(date, time)
    .minusDays(1).minusHours(10).minusSeconds(30);
```
##### Pièges
###### Résultat ignoré
```java
LocalDate date = LocalDate.of(2020, Month.JANUARY, 20);
date.plusDays(10); // résultat ignoré / non assigné à une référence
System.out.println(date); // 2020-01-20
```
###### Types
```java
LocalDate date = LocalDate.of(2020, Month.JANUARY, 20);
date = date.plusMinutes(1); // NE COMPILE PAS
```
Une LocalDate ne contient pas l'heure (*time*). On ne peut pas lui ajouter de minutes.

||Peut être appelé sur `LocalDate`|Peut être appelé sur `LocalTime`|Peut être appelé sur `LocalDateTime` ou `ZonedDateTime`|
|---|:---:|:---:|:---:|
|`plusYears` / `minusYears`|✔|❌|✔|
|`plusMonths` / `minusMonths`|✔|❌|✔|
|`plusWeeks` / `minusWeeks`|✔|❌|✔|
|`plusDays` / `minusDays`|✔|❌|✔|
|`plusHours` / `minusHours`|❌|✔|✔|
|`plusMinutes` / `minusMinutes`|❌|✔|✔|
|`plusSeconds` / `minusSeconds`|❌|✔|✔|
|`plusNanos` / `minusNanos`|❌|✔|✔|

### Travailler avec les périodes
#### Convertir vers un long 
LocalDate et LocalDateTime ont une méthode pour se convertir vers un long
- `LocalDate` a la méthode `toEpochDay()` qui est le nombre de jours depuis le premier janvier 1970,
- `LocalDateTime` et `ZonedDateTime` ont la méthode `toEpochSeconds()` qui est le nombre de secondes depuis le premier janvier 1970,
- `LocalTime` n'a pas de telle méthode car elle représente une heure (*time*) de n'importe quelle date.

#### Exemple de périodes
```java
// Exemple sans les périodes
public static void main(String[] args) { 
    LocalDate start = LocalDate.of(2015, Month.JANUARY, 1);
    LocalDate end = LocalDate.of(2015, Month.MARCH, 30);
    performAnimalEnrichment(start, end);
}

private static void performAnimalEnrichment(LocalDate start, LocalDate end) {
    LocalDate upTo = start;
    while (upTo.isBefore(end)) {
        System.out.println("Give new toy : " + upTo);
        upTo= upTo.plusMonth(1);        // ajoute un mois
    }
}
``` 
Le problème de cette approche est qu'elle n'est pas réutilisable. En utilisant les périodes on peut :
```java
// Exemple avec les périodes
public static void main(String[] args) { 
    LocalDate start = LocalDate.of(2015, Month.JANUARY, 1);
    LocalDate end = LocalDate.of(2015, Month.MARCH, 30);
    Period period = Period.ofMonths(1);  // Crée une période
    performAnimalEnrichment(start, end, period);
}

private static void performAnimalEnrichment(LocalDate start, LocalDate end, Period period) {        // Utilise les périodes
    LocalDate upTo = start;
    while (upTo.isBefore(end)) {
        System.out.println("Give new toy : " + upTo);
        upTo= upTo.plus(period);        // Ajoute la période
    }
}
```
Maintenant on peut choisir la période que l'on veut.

#### Créer des périodes
Il y a 5 façons de créer des périodes :
```java
Period annualy = Period.ofYears(1);             // tous les ans
Period quarterly = Period.ofMonths(3);          // tous les 3 mois
Period everyThreeWeeks = Period.ofWeeks(3);     // toutes les 3 semaines
Period everyOtherDay = Period.ofDays(2);        // tous les 2 jours
Period everyYearAndAWeek = Period.of(1, 0, 7);  // tous les ans et 7 jours
```

Vous ne pouvez pas chaîner les méthodes avec les périodes car les méthodes `of___` sont statiques.
```java
// On ne peut pas chainer les méthodes avec les périodes
Period wrong = Period.ofYears(1).ofWeeks(1);    // Toutes les semaines -- ofYear ignoré
```
#### Affichage de période
```
    P       1Y      2M      3D
    |       |       |       |
Période    #années #mois   #jours
Obligatoire
```
Java omet toutes les mesures égales à zéro.
```java
System.out.println(Period.ofMonths(3)); // P3M -- Année et jour ignorés
System.out.println(Period.of(0, 20, 47)); // P20M47D -- Année ignorée
System.out.println(Period.ofWeeks(3)); // P21D -- 3 semaines = 21 jours
```
#### Piège
```java
LocalDate date = LocalDate.of(2015, 1, 20);
LocalTime time = LocalTime.of(6, 15);
LocalDateTime dateTime = LocalDateTime.of(date, time);
Period period = Period.ofMonths(1);

System.out.println(date.plus(period));      // 2015-02-20
System.out.println(datetime.plus(period));  // 2015-02-20T06:15
System.out.println(date.plus(period));      // UnsupportedTemporalTypeException
```
### Travailler avec les durées
Une *période* est pour un jour ou plus. Une *durée* est prévue pour des unités de temps plus petite.

Pour `Duration` vous pouvez passer le nombre de jours, heures, minutes, secondes, nanosecondes.

`Duration` marche a peu près comme `Period` sauf qu'elle est utilisée avec des objets qui ont l'heure (*time*).

Souvenez vous que les `Period` sont affichées en commençant par un `P`. Les `Duration` commencent par `PT` (« *period of time* »).

Les `Duration` sont stockées en heures, minutes, secondes (le nombre de secondes peut être une fraction) :
```java
Duration daily = Duration.ofDays(1);                // PT24H
Duration hourly = Duration.ofHours(1);              // PT1H
Duration everyMinute = Duration.ofMinutes(1);       // PT1M
Duration everyTensSeconds = Duration.ofSeconds(10); // PT10S
Duration everyMilli = Duration.ofMillis(1);         // PT0.001S
Duration everyNano = Duration.ofNanos(1);           // PT0.000000001S
```
`Duration` n'a pas de constructeur qui prend différentes unités. Pour passer une heure et demie vous devez passer 90 minutes.

`Duration` inclut aussi une *factory method* générique. Elle prend un *number* et une `TemporalUnit`.

`TemporalUnit` est une interface. Pour le moment, il n'y a qu'une seule implémentation appelée `ChronoUnit`.
```java
Duration daily = Duration.of(1, ChronoUnit.DAYS); 
Duration hourly = Duration.ofHours(1, ChronoUnit.HOURS); 
Duration everyMinute = Duration.ofMinutes(1, ChronoUnit.MINUTES); 
Duration everyTensSeconds = Duration.ofSeconds(10, ChronoUnit.SECONDS); 
Duration everyMilli = Duration.ofMillis(1, ChronoUnit.MILLIS); 
Duration everyNano = Duration.ofNanos(1, ChronoUnit.NANOS); 
```
`ChronoUnit` possède aussi des unités pratiques comme `ChronoUnit.HALF_DAYS` pour représenter 12 heures.

`ChronoUnit` est pratique pour savoir combien de temps il y a entre deux valeurs :
```java
LocalTime one = LocalTime.of(5, 15);
LocalTime two = LocalTime.of(6, 30);
LocalDate date = LocalDate.of(2016, 1, 20);

System.out.println(ChronoUnits.HOURS.between(one, two)); // 1
System.out.println(ChronoUnits.MINUTES.between(one, two)); // 75
System.out.println(ChronoUnits.MINUTES.between(one, date)); // DateTimeException
```
Utiliser les `Duration` mace comme utiliser les `Period`.
```java
// Exemple en ajoutant 6 heures
LocalDate date = LocalDate.of(2015, 1, 20);
LocalTime time = LocalTime.of(6, 15);
LocalDateTime dateTime = LocalDateTime.of(date, time);

Duration duration = Duration.ofHours(6);

System.out.println(dateTime.plus(duration)); // 2015-01-20T12:15
System.out.println(time.plus(duration)); // 12:15
System.out.println(date.plus(duration)); // UnsupportedTemporalException
```
```java
// Exemple en ajoutant 23 heures
LocalDate date = LocalDate.of(2015, 1, 20);
LocalTime time = LocalTime.of(6, 15);
LocalDateTime dateTime = LocalDateTime.of(date, time);

Duration duration = Duration.ofHours(23);

System.out.println(dateTime.plus(duration)); // 2015-01-21T05:15
System.out.println(time.plus(duration)); // 05:15
System.out.println(date.plus(duration)); // UnsupportedTemporalException
```
Souvenez-vous que les `Period` et les `Duration` ne sont pas équivalentes.
```java
// Exemple de Period et Duration de même taille
LocalDate date = LocalDate.of(2015, 5, 25);

Period period = Period.ofDays(1);
Duration days = Duration.ofDays(1);

System.out.println(date.plus(period)); // 2015-05-26
System.out.println(date.plus(days)); // Unsupported unit: Seconds
```

||Peut être utilisé avec `Period`|Peut être utiliser avec `Duration`|
|---|:---:|:---:|
|`LocalDate`|✔|❌|
|`LocalDateTime`|✔|✔|
|`LocalTime`|❌|✔|
|`ZonedDateTime`|✔|✔|

### Travailler avec les instants
La classe `Instant` représente un moment spécifique dans la timezone GMT
```java
// Exemple d'utilisation d'Instant
Instant now = Instant.now();
// quelque chose qui prend du temps;
Instant later = Instant.now();

Duration duration = Duration. between(now, later);
System.out.println(duration.toMillis()); // 1025
```
```java
// Exemple de création d'Instant à partir d'une ZonedDateTime
LocalDate date = LocalDate.of(2015, 5, 25);
LocalTime time = LocalTime.of(11, 55, 00);
ZoneId zone = ZoneId.of("US/Eastern")
ZonedDateTime zonedDateTime = LocalDateTime.of(date, time, zone);

Instant instant = zonedDateTime.toInstant(); 

System.out.println(zonedDateTime); // 2015-05-25T11:55-04:00[US/Eastern]
System.out.println(instant); // 2015-05-25T11:55:00Z
```
Vous ne pouvez pas convertir une `LocalDateTime` vers un `Instant`. Un `Instant` est un moment dans le temps. Puisqu'une `LocalDateTime` ne contient pas de timezone elle n'est pas considérée comme le même moment partout dans le monde.
```java
// Exemple de création d'Instant à partir du nombre de secondes depuis le premier janvier 1970
Instant instant = Instant.ofEpochSecond(epochSecond);
System.out.println(instant); // 2015-05-25T11:55:00Z
```

En utilisant cet `Instant` vous pouvez faire des maths.

> Les `Instant` vous permettent d'ajouter avec toutes les unités plus petites que le jour (inclue).
```java
Instant nextDay = Instant.plus(1, ChronoUnit.DAYS);
System.out.println(nextDay); // 2015-05-26T15:55:00Z

Instant nextHour = Instant.plus(1, ChronoUnit.HOURS);
System.out.println(nextHour); // 2015-05-26T16:55:00Z

Instant nextWeek = Instant.plus(1, ChronoUnit.WEEKS); // exception 
```

### Prendre en compte le changement d'heure
On avance d'une heure en mars et on recule d'une heure en novembre.
```
Jour normal                 -- 1:00AM -> 1:59AM -> 2:00AM -> 3:00AM -> 4:00AM ...
Changement d'heure mars     -- 1:00AM -> 1:59AM -> 3:00AM ...
Changement d'heure novembre -- 1:00AM -> 1:59AM -> 1:00AM (Encore) -> 1:59AM ...
```

L'offset UTC change lors du changement d'heure
```java
// Exemple de changement d'heure (mars)
LocalDate date = LocalDate.of(2016, Month.MARCH, 13);
LocalTime time = LocalTime.of(1, 30, 00);
ZoneId zone = ZoneId.of("US/Eastern")
ZonedDateTime dateTime = LocalDateTime.of(date, time, zone);

System.out.println(dateTime); // 2016-03-13T01:30-05:00[US/Eastern]

dateTime = dateTime.plusHours(1); 

System.out.println(dateTime); // 2016-03-13T03:30-04:00[US/Eastern]
```
```java
// Exemple de changement d'heure (novembre)
LocalDate date = LocalDate.of(2016, Month.NOVEMBER, 6);
LocalTime time = LocalTime.of(1, 30, 00);
ZoneId zone = ZoneId.of("US/Eastern")
ZonedDateTime dateTime = LocalDateTime.of(date, time, zone);

System.out.println(dateTime); // 2016-11-06T01:30-04:00[US/Eastern]

dateTime = dateTime.plusHours(1); 

System.out.println(dateTime); // 2016-11-06T01:30-05:00[US/Eastern]

dateTime = dateTime.plusHours(1); 

System.out.println(dateTime); // 2016-11-06T02:30-05:00[US/Eastern
```
```java
// Exemple de date spécifié pendant le changment d'heure
// Java change d'offset automatiquement
LocalDate date = LocalDate.of(2016, Month.MARCH, 13);
LocalTime time = LocalTime.of(2, 30, 00); // -- il n'y a pas de 2:30AM ce jour là
ZoneId zone = ZoneId.of("US/Eastern")
ZonedDateTime dateTime = LocalDateTime.of(date, time, zone);

System.out.println(dateTime); // 2016-03-13T03:30-04:00[US/Eastern]
```

### Examination de la classe `String`
La classe `String` est `final` et les objets `String` sont immutables. Vous ne pouvez pas modifier un objet immutable.

Cela permet à Java d'optimiser et de mettre les *littéraux string* dans la *string pool*.
#### Piège - Comparaison
Pour comparer des `String` : 
- Si vous utilisez `==`, il vérifiera que les références pointent sur le même objet
- Si vous utilisez `equals()`, il vérifiera que le contenu est le même

```java
// exemple d'utilisation de == et equals avec les String
String s1= "bunny";
String s2= "bunny";
String s3= new String("bunny");

System.out.println(s1 == s2);       // true - pointent vers le même littéral dans la string pool
System.out.println(s1 == s3);       // false - pointe vers deux objets différents
System.out.println(s1.equals(s3));  // true - même valeur
``` 
#### Piège - Concaténation avec des chiffres
```java
String s4 = "1" + 2 + 3;
String s5 = 1 + 2 + "3";

System.out.println(s4); // 123
System.out.println(s5); // 33
```

#### Méthodes communes
```java
// Exemple de méthodes communes de String
String s = "abcde ";

System.out.println(s.trim().lenght());                  // 5
System.out.println(s.charAt(4));                        // e
System.out.println(s.indexOf('e'));                     // 4
System.out.println(s.indexOf("de"));                    // 3
System.out.println(s.substring(2, 4).toUpperCase());    // CD
System.out.println(s.replace('a', '1'));                // 1bcde 
System.out.println(s.contains("DE"));                   // false
System.out.println(s.startsWith("a"));                  // true
```
#### `StringBuilder`
Un `StringBuilder` est *mutable*. On peut le modifier ou accroître sa capacité.

Si plusieurs *threads* vont mettre à jour le même objet, vous devriez utiliser `StringBuffer` plutôt que `StringBuilder`.

```java
// Exemple d'utilisation de StringBuilder
StringBuilder b = new StringBuilder();
b.append(12345).append('-');

System.out.println(b.length());     // 6
System.out.println(b.indexOf("-")); // 5
System.out.println(b.charAt(2));    // 3

StringBuilder b2 = b.reverse();

System.out.println(b.toString());   // -54321 
System.out.println(b == b2);        // true
```
```java
StringBuilder s = new StringBuilder("abcde");
s.insert(1, '-').delete(3, 4);

System.out.println(s);                  // a-bde
System.out.println(s.substring(2, 4));  // bd -- index 2 (inclus) à 4 (exclus)
```


|Caractéristiques|`String`|`StringBuilder`|`StringBuffer`|
|---|:---:|:---:|:---:|
|Immutable ?|✔|❌|❌|
|*pooled* ?|✔|❌|❌|
|*thread-safe* ?|✔|❌|✔|
|Peut changer de taille ?|❌|✔|✔|
