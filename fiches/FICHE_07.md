# Programmation Fonctionnelle
## Utiliser des variables dans les lambdas
Les lambdas ont les mêmes règles que les *inner classes*.

Les expressions lambdas peuvent utiliser des :
- variables statiques,
- variables d'instance,
- des paramètres de méthode *effectively final*,
- des variables locales *effectively final*.

Si une variable est réassignée, elle n'est plus *effectively final* et son utilisation dans une lambda causera une erreur à la compilation.

Les règles normales de contrôle d'accès s'appliquent toujours, par exemple, une lambda ne peut pas accéder a une variable privée d'une autre classe.
## Travailler avec les interfaces fonctionnelles intégrées
La convention est d'utiliser : 
- `T` comme premier paramètre,
- `U`, si un second paramètre est nécessaire,
- `R` pour « *return* » comme type générique de la valeur retour.

|Interface fonctionnelle|Nombre de paramètres|Type de retour|Nom de la seule méthode abstraite|
|---|---|---|---|
|`Supplier<T>`|0|`T`|`get`|
|`Consumer<T>`|1 (`T`)|`void`|`accept`|
|`BiConsumer<T, U>`|2 (`T`, `U`)|`void`|`accept`|
|`Predicate<T>`|1(`T`)|`boolean`|`test`|
|`BiPredicate<T, U>`|2 (`T`, `U`)|`boolean`|`test`|
|`Function<T, R>`|1 (`T`)|`R`|`apply`|
|`BiFunction<T, U, R>`|2 (`T`, `U`)|`R`|`apply`|
|`UnaryOperator<T>`|1 (`T`)|`T`|`apply`|
|`BinaryOperator`|2(`T`, `T`)|`T`|`apply`|

Plusieurs autres interfaces fonctionnelles sont définies dans le package `java.util.function`, vous les verrez plus tard dans ce chapitre.

La plupart de temps non n'assignons pas l'implémentation de l'interface à une variable.

> Vous devez apprendre ce tableau

Vous pouvez nommer une interface fonctionnelle comme vous voulez, du moment que c'est un nom d'interface valide et qu'elle ne doit contenir qu'une seule méthode abstraite

### Implémenter `Supplier`
Un `Supplier` est utilisé lorsque vous souhaitez générer ou fournir (*supply*) des valeurs sans prendre d'arguments en entrée.
```java
@FunctionalInterface
public class Supplier<T> {
    public T get();
}
```

```java
// Exemple d'utilisation
Supplier<LocalDate> s1 = LocalDate::now;
Supplier<LocalDate> s2 = () -> LocalDate.now();

LocalDate d1 = s1.get();
LocalDate d2 = s2.get();

System.out.println(d1);
System.out.println(d2);
```

Un `Supplier` est souvent utilisé pour créer de nouveaux objets.
```java
// Exemple de création d'objet avec Supplier
Supplier<LocalDate> s1 = StringBuilder::new;
Supplier<LocalDate> s2 = () -> new StringBuilder();

System.out.println(s1.get());
System.out.println(s2.get());
```

Si on essaie d'afficher une lambda, on obtient quelque chose comme : `functionalinterface.BuiltIns$$Lambda$1/791452441@1fb3ebeb` (Le nom de la classe de test est `BuiltIns`, `$$` veut dire que la classe n'existe pas dans un fichier du système de fichier, elle n'existe qu'en mémoire, le reste importe peu)

### Implémenter `Consumer` et `BiConsumer`
Un `Consumer` est utilisé pour faire quelque chose avec un paramètre et ne rien retourner.

La même chose s'applique au `BiConsumer` sauf qu'il prend deux paramètres.

En omettant les méthodes par défaut, ces interfaces se présentent comme ceci :
```java
@FunctionalInterface
public class Consumer<T> {
    void accept(T t);
}

@FunctionalInterface
public class BiConsumer<T, U> {
    void accept(T t, U u);
}
```

Vous avez utilisé `Consumer` dans le chapitre 3 avec `foreach`.
```java
// Exemple d'utilisation de Consumer
Consumer<String> c1 = System.out::println;
Consumer<String> c2 = System.out.println();

c1.accept("Annie");
c2.accept("Annie");
```

```java
// Exemple d'utilisation de BiConsumer avec deux paramètres de types différents
Map<String, Integer> map = new HashMap<>();
BiConsumer<String, Integer> b1 = map::put;
BiConsumer<String, Integer> b2 = (k, v) -> map.put(k, v);

b1.accept("chicken", 7);
b1.accept("chick", 1);

System.out.println(map); // { chicken: 7, chick: 1 }
```

```java
// Exemple d'utilisation de BiConsumer avec deux paramètres de même types
Map<String, Integer> map = new HashMap<>();
BiConsumer<String, String> b1 = map::put;
BiConsumer<String, String> b2 = (k, v) -> map.put(k, v);

b1.accept("chicken", "Cluck");
b1.accept("chick", "Tweep");

System.out.println(map); // { chicken: Cluck, chick: Tweep }
```
### Implémenter `Predicate` et `BiPredicate`
Vous avez utilisé `Predicate` dans l'OCA et plus récemment avec `removeIf`.

`Predicate` est souvent utilisé pour filtrer ou apparier (*matching*).

Un `BiPredicate` est comme un `Predicate` sauf qu'il accepte dux paramètres au lieu d'un.

En omettant les méthodes par défaut, ces interfaces se présentent comme ceci :
```java
@FunctionalInterface
public class Predicate<T> {
    boolean test(T t);
}

@FunctionalInterface
public class BiPredicate<T, U> {
    boolean test(T t, U u);
}
```
Vous pouvez tester une condition grace à `Predicate` :
```java
Predicate<String> p1 = String::isEmpty;
Predicate<String> p2 = x -> x.isEmpty;

System.out.println(p1.test("")); // true
System.out.println(p2.test("")); // true
```
Avec `BiPredicate` :
```java
BiPredicate<String, String> b1 = String::startsWith;
BiPredicate<String, String> b2 = (string, prefix) -> string.startsWith(prefix);

System.out.println(b1.test("chicken", "chick")); // true
System.out.println(b2.test("chicken", "chick")); // true
```
La référence combine deux techniques, `startsWith()` est une méthode d'instance. Cela veut dire que :
- le premier paramètre est utilisé comme l'instance sur laquelle la méthode est appelée,
- le second paramètre est le paramètre de la méthode `startsWith()`

#### Méthodes par défaut des interfaces fonctionnelles
Par définition, les interfaces fonctionnelles ont une seule méthode abstraite.

Or elles ne sont pas obligées de posséder une seule méthode.

Plusieurs des interfaces fonctionnelles communes possèdent des méthodes `default`.

Supposons :
```java
Predicate<String> egg = s -> s.contains("egg");
Predicate<String> brown = s -> s.contains("brown");
```
On pourrait :
```java
Predicate<String> brownEggs = s -> s.contains("egg") && s.contains("brown");
Predicate<String> otherEggs = s -> s.contains("egg") && ! s.contains("brown");
```
Mais il est préférable de :
```java
Predicate<String> brownEggs = egg.and(brown);
Predicate<String> otherEggs = egg.and(brown.negate());
```
### Implémenter `Function` et `BiFunction`
Une `Function` est responsable de change un paramètre d'un certain type en une valeur d'un type potentiellement différent et la retourne.

De la même manière, une `BiFonction` change deux paramètres en une valeur et la retourne.

En omettant les méthodes par défaut, ces interfaces se présentent comme ceci :
```java
@FunctionalInterface
public class Function<T, R> {
    R apply(T t);
}

@FunctionalInterface
public class BiFunction<T, U, R> {
    R apply(T t, U u);
}
```
```java
// Exemple, cette fonction convertit une String vers un int autoboxé en Integer
Function<String, Integer> f1 = String::length;
Function<String, Integer> f2 = x -> x.length();

System.out.println(f1.apply("cluck")); // 5
System.out.println(f2.apply("cluck")); // 5
```
Les paramètres n'ont pas à être de types différents
```java
// Exemple, cette fonction convertit deux String vers une String
BiFunction<String, String, String> b1 = String::concat;
BiFunction<String, String, String> b2 = (string, toAdd) -> string.concat(toAdd);

System.out.println(b1.apply("baby", "chick")); // baby chick
System.out.println(b2.apply("baby", "chick")); // baby chick
```
#### Créer vos propres interfaces fonctionnelles
Java fournit les interfaces pour une et deux parametres, vous pouvez bien sur créer vos propres interfaces fonctionnelles :
```java
@FunctionalInterface
interface myTriFunction<T, U, V, R> {
    R apply(T t, U u, V v);
}
```
### Implémenter UnaryOperator et BinaryOperator
`UnaryOperator` et `BinaryOperator` sont des cas spéciaux d'une fonction.

Ils nécessitent que tous les paramètres soient du même type.

Un `UnaryOperator` transforme sa valeur en une valeur du même type (eg. incrémenter une valeur par un est une opération binaire).

`UnaryOperator` étend `Function`.

Un `BinaryOperator` combine deux valeurs en uen du même type.

De la même manière, `BinaryOperator` étend `BiFunction`.

```java
@FunctionalInterface
public class UnaryOperator<T> extends Function<T, T> {}
public class BinaryOperator<T> extends BiFunction<T, T, T> {}
```
Les signatures des méthodes ressemblent à :
```java
T apply(T t1);
T apply(T t1, T t2);
``` 
```java
// Exemple UnaryOperator
UnaryOperator<String> u1 = String::toUpperCase;
UnaryOperator<String> u2 = x -> x.toUpperCase();

System.out.println(u1.apply("chirp"); // CHIRP
System.out.println(u2.apply("chirp"); // CHIRP
```

```java
// Exemple BinaryOperator
UnaryOperator<String> b1 = String::concat;
UnaryOperator<String> b2 = (string, toAdd) -> string.concat(toAdd);

System.out.println(b1.apply("baby", "chick"); // baby chick
System.out.println(b2.apply("baby", "chick"); // baby chick
```

Nous n'avons pas besoin de spécifier le type de retour dans les génériques car il est le même que celui des paramètres.

### Exemples des interfaces fonctionnelles
> il est important de connaitre le nombre de paramètres, les types, la valeur de retour et le nom de la méthode pour chaque interface fonctionnelle.

```java
// Exemples de pièges
Function<List<String>> ex1 = x -> x.get(0); // NE COMPILE PAS, Le type de retour n'est pas spécifié
UnaryOperator<Long> ex2 = (Long l) -> 3.14; // NE COMPILE PAS, ne retourne pas le bon type (double au lieu de Long)
Predicate ex4 = String::isEmpty; // NE COMPILE PAS, générique manquant.
```
### Retourner un `Optional`
Depuis Java 8, on peut utiliser le type `Optional`.

Un `Optional` est créé depuis une factory. Vous pouvez soit demander un `Optional` vide ou passer une valeur qui va être enveloppée (*wrapped*) par l'`Optional`.

Pensez aux Optional comme une boîte qui pourrait avoir quelque chose dedans.

```java
// Exemple Optional
public static Optional<Double> average(int... scores) {
    if(scores.length == 0) return Optional.empty();
    int sum = 0;
    for(int score: scores) sum += score;
    return Optional.of((double) sum / scores.length);
}
// Exemple d'utilisation
System.out.println(average(90, 100)); // Optional[95.0]
System.out.println(average()); // Optional.empty
```
```java
// Récupérer les valeurs
Optional<Double> opt = average(90,100);
if(opt.isPresent()) {
    System.out.println(opt.get()); // 95.0
}
```
Si l'`Optional` était vide lors de l'appel à `get`, l'exception `java.util.NoSuchElementException: No value present` serait lancée.

Il est d'usage d'utiliser `empty` quand la valeur est `null`. Il existe deux moyens d'y parvenir :
```java
// via l'instruction if ou une opération ternaire
Optional o = (value == null) ? Optional.empty() : Optional.of(value);

// via la méthode ofNullable fournie par java
Optional o = Optional.ofNullable(value);
```

|Méthode|Quand l'`Optional` est vide|Quand l'`Optional` contient une valeur|
|---|---|---|
|`get()`|Lance une exception|retourne la valeur|
|`ifPresent(Consumer c)`|Ne fait rien|appelle `Consumer  c` avec la valeur| 
|`isPresent()`|Retourne `false`|Retourne `true`|
|`orElse(T other)`|Retourne le paramètre `other`|Retourne la valeur|
|`orElseGet(Supplier s)`|Retourne le résultat de l'appel au `Supplier`|Retourne la valeur|
|`orElseThrow`|Retourne l'exception créée par l'appel au `Supplier`|Retourne la valeur|

```java
// Exemple de piège
System.out.println(opt.orElseGet(
    () -> new IllegalStateException()
));

// Ne compile pas car le paramètre Supplier de la méthode orElseGet doit retourner un Double
// Puisqu'il renvoie une Exception, les types ne correspondent pas.
```
#### `Optional` est-il pareil que `null` ?
Avant Java 8, les développeurs retournaient `null` plutôt qu'`Optional`, il y a quelques lacunes à cette approche :
- Le fait que `null` soit une valeur spéciale n'est pas exprimée clairement,
- On ne peut pas utiliser le style de programmation fonctionnelle d'`Optional` (eg. ifPresent()).