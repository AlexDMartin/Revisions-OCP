## Ajouter l'*internationalization* et la *localization*
L'*internationalization* est le fait de concevoir votre programme pour qu'il soit adapté. Vous n'avez pas à supporter plus d'un langage pour internationaliser votre programme, L'*internationalization* veut juste dire que vous pouvez en supporter d'autres.

La *localization* veut dire que vous supportez différentes *locales*. Oracle définit une *locale* comme "une région géographique, politique, ou culturelle spécifique".

La *localization* inclut : 
- traduire les chaînes de caractère dans un langage différent
- afficher les dates et les nombres dans un format correct pour cette *locale*.

Vous pouvez avoir à passer par le processus de *localization* plusieurs fois pour la même application si vous voulez ajouter des langages et des pays.

*internationalization* et *localization* sont souvent abrégés respectivement *i18n* et *l10n*
### Choisir une *locale*
La classe `Locale` est définie dans le package `java.util`.

La première locale intéressante est la locale actuelle de l'utilisateur :
```java
Locale locale = local.getDefault();
System.out.println(locale)
```
#### Format de la locale
En premier on retrouve le code du langage (eg. `fr`) puis un underscore `_` et enfin le code du pays en majuscule `(eg. `FR`). L'underscore et le code pays sont optionnels

```java
// Exemple de locales invalides
US      // -- on peut avoir un langage sans le pays mais pas l'inverse
enUS    // -- il manque l'underscore
US_en   // -- le pays et le langage sont inversés
EN      // -- le langage doit être en minuscule

// version valide 
en_US
```

Vous pouvez utiliser d'autres locales que celle par défaut.

Il y a trois façons de créer une locale. 

##### Grâce aux constantes de la classe `Locale`
La classe `Locale` fournit des constantes pour les locales les plus utilisées.
```java
System.out.println(Locale.GERMAN);  // de
System.out.println(Locale.GERMANY); // de_DE
```
##### Grâce au constructeur
```java
System.out.println(new Locale("fr"));           // fr
System.out.println(new Locale("hi", "IN"));     // hi_IN
```
##### Grâce au Builder
```java
Locale l1 = new Locale.Builder()
    .setLanguage("en")
    .setRegion("US")
    .build();

Locale l2 = new Locale.Builder()
    .setRegion("US")
    .setLanguage("en")
    .build();
```
###### Piège - Builder
Le builder de `Locale` convertit n majuscule et en minuscule au besoin.

Ce qui veut dire que le code suivant est légal :
```java
Locale l2 = new Locale.Builder()
    .setRegion("us")
    .setLanguage("EN")
    .build();
```
#### Changer de locale
Vous pouvez changer la locale par défaut comme ceci :
```java
System.out.println(Locale.getDefault()); // en_US

Locale locale = new Locale("fr");
Locale.setDefault(locale);

System.out.println(Locale.getDefault()); // fr
```
### Utiliser un *resource bundle*
Un *resource bundle* contient les objets spécifiques à la locale utilisés par le programme.

Il se présente comme une carte avec des clés et des valeurs.

Le *resource bundle* peut être dans une classe java ou dans un fichier de propriétés.

Un fichier de propriétés (*property file*) est un fichier dans un format spécifique avec des paires clé / valeur.

```java
// Locales utilisées dans les exemples
Locale us = new Locale("en", "US");
Locale france = new Locale("fr", "FR");
Locale englishCanada = new Locale("en", "CA");
Locale frenchCanada = new Locale("fr", "CA");
```

### Créer un *resource bundle* via un fichier de propriétés 
Java ne nous oblige pas à créer quatre *resource bundle*. Si nous n'avons pas de *resource bundle* spécifique au pays, java utilisera celui spécifique au langage.
```java
// Zoo_en.properties
hello=Hello
open=The zoo is open

//Zoo_fr.properties
hello=Bonjour
open=Le zoo est ouvert 
```
Le nom des fichiers de propriété est le nom de notre *resource bundle* suivi d'un underscore `_` puis de la locale cible.

On peut écrire notre programme utilisant ce resource bundle :
```java
import java.util.*;

public class ZooOpen {
    
    public static void main(Sting[] args) {
        Locale us = new Locale("en", "US");
        Locale france = new Locale("fr", "FR");
        printProperties(us);
        System.out.println();
        printProperties(fr);
    }
    
    public static void printProperties(Locale locale) {
        ResourceBundle rb = ResourceBundle.getBundle("Zoo", locale);
        System.out.println(rb.getSetting("hello"));
        System.out.println(rb.getSetting("open"));
    }
}

/* Output:
* Hello
* The zoo is open
* 
* Bonjour
* Le zoo est ouvert
*/
```
##### Format du fichier de propriétés
La syntaxe la plus commune est : `animal=dolphin`.

Vous pouvez utiliser `animal:dolphin` ou `animal dolphin`.

Règles principales :
- Si la ligne commence par un `#` ou un `!` c'est un commentaire,
- les espaces situés avant et après le séparateur sont ignorés,
- les espaces situés au début de la ligne sont ignorés,
- les espaces situés à la fin de la ligne **ne sont pas** ignorés,
- finissez une ligne avec un backslash `\` si vous voulez faire un saut de ligne (lisibilité),
- vous pouvez utiliser les caractères d'échappement de Java (eg. `\n` `t`).

```properties
# Exemple de fichier de propriétés
! another comment
key = value\tafter tab
long = abcdefghijklm\
  nopqrstuvwxyz
```
```
value   after tab
abcdefghijklmnopqrstuvwxyz
```
Puisqu'un *resource bundle* contient des paires de clé / valeur, vous pouvez boucler dessus pour lister toutes les paires.

La classe `ResourceBundle` fournit les méthodes pour récupérer un set de toutes les clés.
```java
// Exemple d'utilisation de la classe ResourceBundle
Locale us = new Lcoale("en", "US");
ResourceBundle rb = ResourceBundle.getBundle("Zoo", us);

Set<String> keys = rb.keySet();
keys.stream().map(k -> k + " " + rb.getString(k))
    .forEach(System.out::println);
```
Java supporte aussi l'utilisation de la classe `Properties` (similaire à une map) :
```java
// Exemple de conversion de ResourceBundle à Properties
Properties props = new Properties();
rb.keySet().stream()
    .forEach(k -> props.put(k, rb.getString(k)));
```
```java
// Exemple d'utilisation de getProperty()
System.out.println(props.getProperty("notReallyAProperty"));        // valeur de la propriété notReallyAProperty ou null
System.out.println(props.getProperty("notReallyAProperty", "123")); // valeur de la propriété notReallyAProperty ou 123
```
Valeurs de retour de getProperty :

|Clé trouvée|Oui|Non|
|---|:---:|:---:|
|`getProperty("key")`|Valeur|`null`|
|`getProperty("key", "default")`|Valeur|`"default"`|

### Créer un *resource bundle* via une classe Java 
Pour implémenter un *resource bundle* en jav, vous créez une classe avec le même nom que celui que vous auriez utilisé avec un fichier de propriété, seule l'extension est différente.

Puisque nous avons à faire à un objet Java l'extension doit être `.java` plutôt que `.properties`.
```java
// Exemple du fichier de propriété précédent converti en classe Java
import java.util.*;

public class Zoo_en extends ListResourceBundle {
    protected Object[][] getContents() {
        return new Object[][] {
            {"hello", "Hello"},
            {"open", "The zoo is open"}
        };
    }
}
```
Avantages de cette méthode :
- vous pouvez utiliser des types de valeur qui ne sont pas des `String`,
- Vous pouvez créer des valeurs à l'exécution.

```java
// Exemple de création de valeurs à l'exécution
package resourcebundles;
import java.util.*;

public class Tax_en_US extends ListResourceBundle {
    protected Object[][] getContents() {
        return new Object[][] {{"tax", new UsTaxCode()}};
    }
    
    public static void main(String[] args) {
        ResourceBundle rb = ResourceBundle.getBundle("resourcebundles.Tax", Locale.US);
        System.out.println(rb.getObject("tax")); // on utilise getObject car la valeur n'est pas une String
    }
}
```
### Déterminer quel *resource bundle* utiliser
À l'examen, il y a deux méthodes pour récupérer un *resource bundle* :
```java
ResourceBundle.getBundle("name");           // Utilise la locale par défaut
ResourceBundle.getBundle("name", locale);
```
Java cherche à utiliser la valeur la plus spécifique. Lorsqu'il y a égalité, les *resource bundles* sous forme de classe java sont priorisés

Exemple de recherche de *resource bundle* pour la locale `new Locale("fr, "FR")` lorsque la locale par défaut est en_US :

|Étape|Recherche du fichier|Raison|
|---|:---:|:---:|
|1|`Zoo_fr_FR.java`|Locale demandée|
|2|`Zoo_fr_FR.properties`|Locale demandée|
|3|`Zoo_fr.java`|Langage demandé sans le pays|
|4|`Zoo_fr.properties`|Langage demandé sans le pays|
|5|`Zoo_en_US.java`|Locale par défaut|
|6|`Zoo_en_US.properties`|Locale par défaut|
|7|`Zoo_en.java`|Langage par défaut sans le pays|
|8|`Zoo_en.properties`|Langage par défaut sans le pays|
|9|`Zoo.java`|Pas de locale, bundle par défaut|
|10|`Zoo.properties`|Pas de locale, bundle par défaut|
|11|Si toujours pas trouvé, lance une `MissingResourceException`||

Conseils :
- Recherchez toujours le fichier de propriété après la classe Java correspondante,
- Supprimez une chose à la fois s'il n'y a pas de correspondance, supprimez d'abord le pays puis le langage,
- Recherchez la locale par défaut puis le *resource bundle* par défaut à la fin.

Java n'est pas obligé de recevoir toutes les clés d'un unique *resource bundle*. Il peut les recevoir de n'importe quel parent de ce *resource bundle*.

<table>
<thead>
  <tr>
    <th>Resource bundle correspondant</th>
    <th>Fichiers d'où peuvent venir les clés</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td rowspan="3">Zoo_fr_FR.java</td>
    <td>Zoo_fr_FR.java</td>
  </tr>
  <tr>
    <td>Zoo_fr.java</td>
  </tr>
  <tr>
    <td>Zoo.java</td>
  </tr>
  <tr>
    <td rowspan="2">Zoo_fr.properties</td>
    <td>Zoo_fr.properties</td>
  </tr>
  <tr>
    <td>Zoo.properties</td>
  </tr>
</tbody>
</table>

### Traitement des variables dans un *resource bundle*
Il est commun de substituer des variables au milieu d'un *resource bundle*.

Par convention, on utilise un nombre entre brackets `{0}`.

Les resource bundle ne les supporte pas mais la classe `MessageFormat` si.
```properties
helloByName = Hello, {0}
```
Le second argument de la méthode `format()` est un vararg.
```java
String format = rb.getString("helloByName");
String formatted = MessageFormat.format(format, "Tammy");
System.out.print(formatted); // Hello, Tammy
```
### Formatter des nombres
#### Formatter et parser les nombres et les devises
Sans tenir compte de ce que vous voulez formatter ou parser, la première étape est toujours la même : Vous devez créer un `NumberFormat`.

Cette classe fournit des *factory method* pour obtenir le *formatter* voulu  :

<table>
<thead>
  <tr>
    <th>Description</th>
    <th>En utilisant la Locale par défaut et la Locale spécifiée<br></th>
  </tr>
</thead>
<tbody>
  <tr>
    <td rowspan="2">Un formatter à usage général</td>
    <td><code>NumberFormat.getInstance()</code></td>
  </tr>
  <tr>
    <td><code>NumberFormat.getInstance(locale)</code></td>
  </tr>
  <tr>
    <td rowspan="2">Même chose que getInstance</td>
    <td><code>NumberFormat.getNumberInstance()</code></td>
  </tr>
  <tr>
    <td><code>NumberFormat.getNumberInstance(locale)</code></td>
  </tr>
  <tr>
    <td rowspan="2">Pour formatter des montants monaitaires</td>
    <td><code>NumberFormat.getCurrencyInstance()</code></td>
  </tr>
  <tr>
    <td><code>NumberFormat.getCurrencyInstance(locale)</code></td>
  </tr>
  <tr>
    <td rowspan="2">Pour formatter des pourcentages<br></td>
    <td><code>NumberFormat.getPercentInstance()</code></td>
  </tr>
  <tr>
    <td><code>NumberFormat.getPercentInstance(locale)</code></td>
  </tr>
  <tr>
    <td rowspan="2">Pour arrondir des valeurs décimales avant l'affichage (pas à l'examen)</td>
    <td><code>NumberFormat.getIntegerInstance()</code></td>
  </tr>
  <tr>
    <td><code>NumberFormat.getIntegerInstance(locale)</code></td>
  </tr>
</tbody>
</table>

Une fois que vous avez l'instance de `NumberFormat`, vous pouvez appeler la méthode `format()` pour transformer un nombre en une `String` ou la méthode `parse()` pour transformer une `String` en nombre.
##### Formatter
La méthode `format` formate le nombre en se basant sur la locale associée a l'objet `NumberFormat`

```java
// Exemple de formatage
import java.text.*;
import java.util.*;

public class FormatNumber {
    public static void main(String[] args) {
        int attendeesPerYear = 3_200_000;
        int attendeesPerMonth = attendeesPerYear / 12;
    
        NumberFormat us = NumberFormat.getInstance(Locale.US);
        Sytem.out.println(us.format(attendeesPerMonth)); // 266.666

        NumberFormat g = NumberFormat.getInstance(Locale.GERMANY);
        Sytem.out.println(g.format(attendeesPerMonth)); // 266,666
        
        NumberFormat ca = NumberFormat.getInstance(Locale.CANADA_FRENCH);
        Sytem.out.println(ca.format(attendeesPerMonth)); // 266 666
    }
}
```
On peut formatter des devises de la même manière :
```java
double price = 48;
NumberFormat us = NumberFormat.getCurrencyInstance();
System.out.println(us.format(price)); // $48.00
```
> Dans la vraie vie, on utilise `int` et `BigDecimal` pour la monnaie et non pas `double`. Faire des maths sur des `double` est dangereux.
##### Parser
La classe `NumberFormat` défini une méthode `parse` pour parser une `String` en un nombre en utilisant la locale spécifiée.

La méthode `parse()` pour les différents types de format lance une exception checkée `ParseException` si elle échoue.

Aux états unis, ils utilisent le point `.` comme séparateur pour les décimaux.

En France, on utilise la virgule. 
```java
// Exemple d'utilisation de parse()
NumberFormat en = NumberFormat.getInstance(Locale.US);
NumberFormat fr = NumberFormat.getInstance(Locale.FRANCE);

String s = "40.45";
System.out.println(en.parse(s)); // 40.45
System.out.println(fr.parse(s)); // 40 -- parse s'arrête au point car il n'est pas le séparateur
```

###### Qu'est-ce que java fait des caractères supplémentaires lors du parsing ?
La méthode `parse` ne parse que le début d'une chaîne de caractère. Une fois qu'elle a atteint un caractère qu'elle ne peut pas parser elle s'arrête et renvoie la valeur
```java
NumberFormat nf = NumberFormat.getInstance();
String one = "456abc";
String two = "-2.5165x10";
String three = "x85.3";

System.out.println(nf.parse(one));      // 456
System.out.println(nf.parse(two));      // -2.5165
System.out.println(nf.parse(three));    // lance une ParseException puisqu'il n'y a pas de nombre au début de la string
```

La méthode parse est aussi utilisée pour les devises.
```java
String amt = "$92,807.99";
NumberFormat cf = NumberFormat.getCurrencyInstance();
double value = (Double) cf.parse(amt);
System.out.println(value); // 92807.99
```
#### Formatter les dates et les heures
les dates et les heures supportent beaucoup de méthodes pour leur extraire des données :
```java
LocalDate date = LocalDate.of(2020, Month.JANUARY, 20);

System.out.println(date.getDayOfWeek());    // MONDAY
System.out.println(date.getMonth());        // JANUARY
System.out.println(date.getYear());         // 2020
System.out.println(date.getDayOfYear());    // 20
```
Nous pourrions utiliser cest informations pour les afficher mais java nous fournit la classe `DateTimeFormatter` pour nous y aider.

La classe `DateTimeFormatter` se trouve dans le package `java.time.format`.

```java
// Exemple d'utilisation de DateTimeFormatter - format favorisant les machines
LocalDate date = LocalDate.of(2020, Month.JANUARY, 20);
LocalTime time = LocalTime.of(11, 12, 14);
LocalDateTime dateTime = LocalDateTime(date, time);

System.out.println(date.format(DateTimeFormatter.ISO_LOCAL_DATE));          // 2020-01-20
System.out.println(time.format(DateTimeFormatter.ISO_LOCAL_TIME));          // 11:12:34
System.out.println(dateTime.format(DateTimeFormatter.ISO_LOCAL_DATE_TIME)); // 2020-01-20T11:12:34
```

```java
// Exemple d'utilisation de DateTimeFormatter - format favorisant les humains - format appelé sur le formatter
DateTimeFormatter shortDateTime = DateTimeFormatter.ofLocalizedDate(FormatStyle.SHORT);

System.out.println(shortDateTime.format(dateTime)); // 1/20/20
System.out.println(shortDateTime.format(date));     // 1/20/20
System.out.println(shortDateTime.format(time));     // UnsupportedTemporalTypeException
```

La méthode `format` peut être appelée sur le formatter et sur la date, donc ce code fait la même chose :
```java
// Exemple d'utilisation de DateTimeFormatter - format favorisant les humains - format appelé sur la date
DateTimeFormatter shortDateTime = DateTimeFormatter.ofLocalizedDate(FormatStyle.SHORT);

System.out.println(dateTime.format(shortDateTime)); // 1/20/20
System.out.println(date.format(shortDateTime));     // 1/20/20
System.out.println(time.format(shortDateTime));     // UnsupportedTemporalTypeException
```
##### Utilisation des méthodes `ofLocalized`

|`DateTimeFormatter f = DateTimeFormatter.______(FormatStyle.SHORT)`|Appeler `f.format(localDate)`|Appeler `f.format(localDateTime)` ou `(zonedDateTime)`|Appeler `f.format(localTime)`|
|---|:---:|:---:|:---:|
|`ofLocalizedDate`|Légal, affiche l'objet entier|Légal, affiche uniquement la date|Lance une exception à l'exécution|
|`ofLocalizedDateTime`|Lance une exception à l'exécution|Légal, affiche l'objet entier|Lance une exception à l'exécution|
|`ofLocalizedTime`|Lance une exception à l'exécution|Légal, affiche uniquement l'heure|Légal, affiche l'objet entier|

Il y a deux formats prédéfinis à apprendre pour l'examen, `SHORT` et `MEDIUM` (ceux avec les timezones ne sont pas à l'examen).

```java
// Exemple d'utilisation des formats prédéfinis SHORT et MEDIUM
LocalDate date = LocalDate.of(2020, Month.JANUARY, 20);
LocalTime time = LocalTime.of(11, 12, 34);
LocalDateTime dateTime = LocalDateTime(date, time);

DateTimeFormatter shortF = DateTimeFormatter.ofLocalizedDateTime(Format.SHORT);
DateTimeFormatter mediumF = DateTimeFormatter.ofLocalizedDateTime(Format.MEDIUM);

System.out.println(shortF.format(dateTime));    // 1/20/20 11:12 AM
System.out.println(mediumF.format(dateTime));   // Jan 20, 2020 11:12:34 AM
```

##### Créer ses propres *formatters*
```java
DateTimeFormatter f = DateTimeFormatter.ofPattern("MMMM dd, yyyy, hh:mm");

System.out.println(f.format(dateTime));   // January 20, 2020 11:12
```

- `MMMM` représente le mois, plus il y a de `M` plus Java est verbeux (`M` -> 1, `MM` -> 01, `MMM` -> Jan, `MMMM` -> January),
- `dd` représente le jour du mois, plus il y a de d plus Java est verbeux (`dd` prend en compte les zéros non significatifs),
- `,` si vous voulez afficher une virgule,
- `yyyy` représente l'année (`yy` -> sur 2 chiffres, `yyyy` -> sur 4 chiffres),
- `hh` représente l'heure (`hh` prend en compte les zéros non significatifs)
- `:` si vous voulez afficher deux points,
- `mm` représente les minutes (`m` omet les zéros non significatifs, `mm` prend en compte les zéros non significatifs)