## Travailler avec les primitives
Jusqu'à maintenant, nous avons utilisé de classes `wrapper` quand nous avions besoin que des primitives aillent dans un stream.

On peut transformer :
```java
// Exemple stream normal
Stream<Integer> stream = Stream.of(1, 2, 3);
System.out.println(stream.reduce(0, (s, n) -> s + n));
```
En :
```java
// Exemple stream primitif
Stream<Integer> stream = Stream.of(1, 2, 3);
System.out.println(stream.mapToInt(x -> x).sum());
```

Les *streams primitifs* nous permettent d'appliquer des opérations plus facilement.
```java
// Exemple stream primitif -- calcul de moyenne
IntStream intStream = IntStream.of(1, 2, 3);
OptionalDouble avg = intStream.average();
System.out.println(avg.getAsDouble());
```

### Créer un *stream primitif*
Il existe trois types de *streams primitifs* :
- `IntStream` : utilisé pour les *types primitifs* `int`, `short`, `byte` et `char`,
- `LongStream` : utilisé pour le *type primitif* `long`,
- `DoubleStream` : utilisé pour les *types primitifs* `double` et `float`.

Pourquoi tous les *types primitifs* n'ont pas leur propre *stream primitif* ? Ces trois là sont les plus communs, donc ils ont juste développés ceux-là.

> Quand vous voyez le mot « stream » à l'examen, faites attention aux majuscules / minuscules. 
>
> Avec une majuscule, `Stream` est le nom d'une classe qui contient un type d'`Object`.
>
> Avec une minuscule, « stream » est un concept qui peut être un `Stream`, `DoubleStream`, `IntStream` ou `LongStream`.

La plupart des méthodes pour créer un *stream primitif* sont équivalentes à celles des `Stream` normaux.
```java
// Création d'un stream primitif vide
DoubleStream empty = DoubleStream.empty();

// Création d'un stream primitif avec of(varargs ...)
DoubleStream oneValue = DoubleStream.of(3.14);
DoubleStream varargs = DoubleStream.of(1.0, 1.1, 1.2);
```

Cela marche de la même manière pour tous les types de *stream primitif*.

Vous pouvez aussi utiliser les mêmes méthodes que les `Stream` normaux pour créer des *streams primitifs infinis*.
```java
// Création d'un stream primitif -- generate
DoubleStream random = DoubleStream.generate(Math::random);

// Création d'un stream primitif -- iterate
DoubleStream fractions = DoubleStream.iterate(.5, d -> d / 2);
```

Lorsque nous avons à faire à des primitifs `int` ou `long` il est fréquent de devoir compter :
```java
// plage de données avec iterate()
IntStream count = IntStream.iterate(1, n -> n + 1).limit(5);
count.forEach(System.out.print);        // 12345

// plage de données avec range()
IntStream range = IntStream.range(1, 6);
range.forEach(System.out.print);        // 12345

// plage de données avec rangeClosed()
IntStream rangeClosed = IntStream.range(1, 5);
rangeClosed.forEach(System.out.print);  // 12345
```

La dernière façon de créer un stream et de le mapper à partir d'un autre type de *stream*

|Classe du stream source|Pour créer un `Stream`|Pour créer un `DoubleStream`|Pour créer un `IntStream`|Pour créer un `LongStream`|
|---|:---:|:---:|:---:|:---:|
|`Stream`|`map`|`mapToDouble`|`mapToInt`|`mapToLong`|
|`DoubleStream`|`mapToObj`|`map`|`mapToInt`|`mapToLong`|
|`IntStream`|`mapToObj`|`mapToDouble`|`map`|`mapToLong`|
|`LongStream`|`mapToObj`|`mapToDouble`|`mapToInt`|`map`|

Ils doivent être de types compatibles pour pouvoir être convertis.

```java
// Exemple de conversion avec mapToXxx()
Stream<String> objStream = Stream.of("penguin", "fish");
IntStream intStream = objStream.mapToInt(s -> s.length());
```

Lors du mapping, les interfaces fonctionnelles de correspondance doivent être :

|Classe du *stream* source|Pour créer un `Stream`|Pour créer un `DoubleStream`|Pour créer un `IntStream`|Pour créer un `LongStream`|
|---|:---:|:---:|:---:|:---:|
|`Stream`|`Function`|`ToDoubleFunction`|`ToIntFunction`|`ToLongFunction`|
|`DoubleStream`|`DoubleFunction`|`DoubleUnaryOperator`|`DoubleToIntFunction`|`DoubleToLongFunction`|
|`IntStream`|`IntFunction`|`IntToDoubleFunction`|`IntUnaryOperator`|`IntToLongFunction`|
|`LongStream`|`LongFunction`|`LongToDoubleFunction`|`LongToIntFunction`|`LongUnaryOperator`|

> Vous pouvez aussi créer un *stream primitif* à partir d'un stream en utilisant `flatMapToInt()`, `flatMapToDouble()` ou `flatMapToLong()`

```java
// Exemple d'utilisation de flatMapToXxx()
IntStream ints = list.stream().flatMapToInt(x -> IntStream.of(x));
```
### Utiliser `Optional` avec les * streams primitifs*
```java
IntStream stream = IntStrem.rangeClosed(1, 10);
OptionalDouble optional = stream.average();
```

le type de retour n'est pas l'`Optional` habituel.

Pourquoi ne pas utiliser `Optional<Double>` ? parce que `OptionalDouble` est pour le primitif `double` et Optional<Double> est pour le wrapper `Double`.

```java
optional.ifPresent(System.out::println);
System.out.println(optional.getAsDouble());
System.out.println(optional.orElseGet(() -> Double.NaN));
```
La seule différence notable est que nous utilisons `getAsDouble()` plutôt que `get()`.

Cela met en lumière le fait que l'on utilise un primitif.

`orElseGet()` prend en paramètre un `DoubleSupplier` au lieu d'un `Supplier`.

|   |`OptionalDouble`|`OptionalInt`|`OptionalLong`|
|---|:---:|:---:|:---:|
|Récupérer en tant que primitif|`getAsDouble()`|`getAsInt()`|`getAsLong()`|
|Paramètre de `orElseGet()`|`DoubleSupplier`|`IntSupplier`|`LongSupplier`|
|Type de retour de `max()`|`OptionalDouble`|`OptionalInt`|`OptionalLong`|
|Type de retour de `sum()`|`double`|`int`|`long`|
|Type de retour de `avg()`|`OptionalDouble`|`OptionalInt`|`OptionalLong`|

### Synthèse des statistiques
```java
// Exemple -- renvoyer le max du stream
private static int max(IntStream ints) {
    OptionalInt optional = ints.max();
    return optional.orElseThrow(RuntimeException::new);
}
```
Maintenant essayons de renvoyer la range (`max() - min()`).

`max()` et `min()` sont deux opérations terminales, on ne peut pas les chainer.

On peut donc faire :
```java
private static int range(IntStream ints) {
    IntSummaryStatistics stats = ints.summaryStatistics();
    if(stats.getCount() == 0) throw new RuntimeException();
    return stats.getMax() - stats.getMin();
}
```
les getters de [`IntSummaryStatistics`](https://docs.oracle.com/javase/8/docs/api/java/util/IntSummaryStatistics.html) sont :
- `double getAverage()`
- `long getCount()`
- `int getMax()`
- `int getMin()`
- `long getSum()`

### Apprendre les interfaces fonctionnelles des primitives
#### Interface fonctionnelles pour `boolean`
`BooleanSupplier` est un type à part. Il a une méthode à implémenter :
```java
boolean getAsBoolean()
```

```java
// Exemple d'utilisation de BooleanSupplier
BooleanSupplier b1 = () -> true;
BooleanSupplier b2 = () -> Math.random() > .5;

System.out.println(b1.getAsBoolean());
System.out.println(b2.getAsBoolean());
```
#### Interfaces fonctionnelles pour `double`, `int` et `long`

|Interface Fonctionnelle|Nombre de paramètres|Type de retour|Unique méthode abstraite|
|---|:---:|:---:|:---:|
|`DoubleSupplier`|0|`double`|`getAsDouble`|
|`IntSupplier`|0|`int`|`getAsInt`|
|`LongSupplier`|0|`long`|`getAsLong`|
|`DoubleConsumer`|1 (`double`)|`void`|`accept`|
|`IntConsumer`|1 (`int`)|`void`|`accept`|
|`LongConsumer`|1 (`long`)|`void`|`accept`|
|`DoublePredicate`|1 (`double`)|`boolean`|`test`|
|`IntPredicate`|1 (`int`)|`boolean`|`test`|
|`LongPredicate`|1 (`long`)|`boolean`|`test`|
|`DoubleFunction<R>`|1 (`double`)|`R`|`apply`|
|`IntFunction<R>`|1 (`int`)|`R`|`apply`|
|`LongFunction<R>`|1 (`long`)|`R`|`apply`|
|`DoubleUnaryOperator`|1 (`double`)|`double`|`applyAsDouble`|
|`IntUnaryOperator`|1 (`int`)|`int`|`applyAsInt`|
|`LongUnaryOperator`|1 (`long`)|`long`|`applyAsLong`|
|`DoubleBinaryOperator`|2 (`double`, `double`)|`double`|`applyAsDouble`|
|`IntBinaryOperator`|2 (`int`, `int`)|`double`|`applyAsInt`|
|`LongBinaryOperator`|2 (`long`, `long`)|`double`|`applyAsLong`|

Il y a quelques changements par rapport àux interfaces fonctionnelles « normales » :
- Les génériques sont enlevés de certaines interfaces puisque le nom nous dit quel type primitif est utilisé. Dans certains cas comme `IntFunction` seul le générique du type de retour est nécessaire,
- Le nom de la méthode abstraite est souvent, mais pas tout le temps, renommé pour correspondre au type primitif utilisé,
- `BiConsumer`, `BiPredicate` et `BiFunction` n'y sont pas puisque les développeurs de l'API n'ont développé que les opérations les plus utilisées.

Certaines interfaces sont spécifiques aux primitives :

|Interface Fonctionnelle|Nombre de paramètres|Type de retour|Unique méthode abstraite|
|---|:---:|:---:|:---:|
|`ToDoubleFunction<T>`|1 (`T`)|`double`|`applyAsDouble`|
|`ToIntFunction<T>`|1 (`T`)|`int`|`applyAsInt`|
|`ToLongFunction<T>`|1 (`T`)|`long`|`applyAsLong`|
|`ToDoubleBiFunction<T, U>`|2 (`T`, `U`)|`double`|`applyAsDouble`|
|`ToIntBiFunction<T, U>`|2 (`T`, `U`)|`int`|`applyAsInt`|
|`ToLongBiFunction<T, U>`|2 (`T`, `U`)|`long`|`applyAsLong`|
|`DoubleToIntFunction`|1 (`double`)|`int`|`applyAsInt`|
|`DoubleToLongFunction`|1 (`double`)|`long`|`applyAsLong`|
|`IntToDoubleFunction`|1 (`int`)|`double`|`applyAsDouble`|
|`IntToLongFunction`|1 (`int`)|`long`|`applyAsLong`|
|`LongToDoubleFunction`|1 (`long`)|`double`|`applyAsDouble`|
|`LongToIntFunction`|1 (`long`)|`int`|`applyAsInt`|
|`ObjDoubleConsumer<T>`|2 (`T`, `double`)|`void`|`accept`|
|`ObjIntConsumer<T>`|2 (`T`, `int`)|`void`|`accept`|
|`ObjLongConsumer<T>`|2 (`T`, `long`)|`void`|`accept`|

## Travailler avec des concepts avancés de *stream pipeline*
### Relier les *streams* aux données sous-jacentes
```java
List<String> cats = new ArrayList<>();
cats.add("Annie");
cats.add("Rippley");
Stream<String> stream = cats.stream();
cats.add("KC");
System.out.println(stream.count()); // 3
```
### Chaîner les `Optional`
#### Exemple de chaîne basique
Prenons :
```java
private static void threeDigit(Optional<Integer> optional) {
    if(optional.isPresent()) {              // outer if
        integer num = optional.get();
        String string = "" + num;
        if(string,length == 3) {            // inner if
            System.out.println(string)
        }   
    }
}
```

Avec la programmation fonctionnelle on peut :
Prenons :
```java
private static void threeDigit(Optional<Integer> optional) {
   optional.map(n -> "" + n)
        .filter(s -> s.length() == 3)
        .ifPresent(System.out::println);
}
```

#### Exemple des avantages de flatMap 
```java
Optional<Integer> result = optional.map(String::length);
```

Supposons que l'on souhaite ajouter de la logique en appelant une méthode utilitaire `Optional<Integer> calculator(String s)` :

```java
Optional<Integer> result = optional.map(ChainingOptionals::calculator); // NE COMPILE PAS
```

Le problème est que le résultat de l'appel à `map()` renvoit un `Optional<Optional<Integer>>` et de ce fait ne correspond pas au type de `result`.

Or on peut :
```java
Optional<Integer> result = optional.flatMap(ChainingOptionals::calculator); 
```
Celui là marche car il supprime la couche inutile.

Chaîner les appels à `flatMap()` est utile quand vous souhaitexz transformer un type d'optional en un autre.

### Collecter les résultats

<table>
<thead>
  <tr>
    <th>Collecteur</th>
    <th>Description</th>
    <th>Valeur de retour lorsque passé a <code>Collect</code></th>
  </tr>
</thead>
<tbody>
  <tr>
    <td><code>averagingDouble(ToDoubleFunction f)</code></td>
    <td rowspan="3">Calcule la moyenne pour les trois principaux types primitifs </td>
    <td rowspan="3"><code>Double</code></td>
  </tr>
  <tr>
    <td><code>averagingInt(ToIntFunction f)</code></td>
  </tr>
  <tr>
    <td><code>averagingLong(ToLongFunction f)</code></td>
  </tr>
  <tr>
    <td><code>counting()</code></td>
    <td>Compte le nombre d'éléments</td>
    <td><code>Long</code></td>
  </tr>
  <tr>
    <td><code>groupingBy(Function f)</code></td>
    <td rowspan="3">Crée une map regroupée grâce à la fonction spécifiée avec le type optionnel et le collecteur en aval optionnel (optional downstream collector)</td>
    <td rowspan="3"><code>Map&lt;K, List&lt;T&gt;&gt;</code></td>
  </tr>
  <tr>
    <td><code>groupingBy(Function f, Collector dc)</code></td>
  </tr>
  <tr>
    <td><code>groupingBy(Function f, Collector dc)</code></td>
  </tr>
  <tr>
    <td><code>joining()</code></td>
    <td rowspan="2">Crée une unique String en utilisant cs comme délimiteur si spécifié</td>
    <td rowspan="2"><code>String</code></td>
  </tr>
  <tr>
    <td><code>joining(CharSequence cs)</code></td>
  </tr>
  <tr>
    <td><code>maxBy(Comparator c)</code></td>
    <td rowspan="2">Trouve le plus grand / plus petit élément</td>
    <td rowspan="2"><code>Optional&lt;T&gt;</code></td>
  </tr>
  <tr>
    <td><code>minBy(Comparator c)</code></td>
  </tr>
  <tr>
    <td><code>mapping(Function f, Collector dc)</code></td>
    <td>Ajoute un nouvelle couche de collecteur</td>
    <td><code>Collector</code></td>
  </tr>
  <tr>
    <td><code>partitioningBy(Predicate p)</code></td>
    <td rowspan="2">Crée une map regroupée par le Predicate spécifié avec le collecteur en aval optionnel (optional further downstream collector)</td>
    <td rowspan="2"><code>Map&lt;Boolean, List&lt;T&gt;&gt;</code></td>
  </tr>
  <tr>
    <td><code>partitioningBy(Predicate p, Collector dc)</code></td>
  </tr>
  <tr>
    <td><code>summarizingDouble(ToDoubleFunction f)</code></td>
    <td rowspan="3">calcule la moyenne, minimum, maximum etc.</td>
    <td><code>DoubleSummaryStatistics</code></td>
  </tr>
  <tr>
    <td><code>summarizingInt(ToIntFunction f)</code></td>
    <td><code>IntSummaryStatistics</code></td>
  </tr>
  <tr>
    <td><code>summarizingLong(ToLongFunction f)</code></td>
    <td><code>LongSummaryStatistics</code></td>
  </tr>
  <tr>
    <td><code>summingDouble(ToDoubleFunction f)</code></td>
    <td rowspan="3">Calcule la somme pour nos trois types primitifs principaux</td>
    <td><code>Double</code></td>
  </tr>
  <tr>
    <td><code>summingInt(ToIntFunction f)</code></td>
    <td><code>Integer</code></td>
  </tr>
  <tr>
    <td><code>summingLong(ToIntFunction f)</code><br></td>
    <td><code>Long</code></td>
  </tr>
  <tr>
    <td><code>toList()</code></td>
    <td rowspan="2">Crée un type arbitraire de liste ou de set</td>
    <td><code>List</code></td>
  </tr>
  <tr>
    <td><code>toSet()</code></td>
    <td><code>Set</code></td>
  </tr>
  <tr>
    <td><code>toCollection(Supplier s)</code></td>
    <td>Crée une Collection du type spécifié</td>
    <td><code>Collection</code></td>
  </tr>
  <tr>
    <td><code>toMap(Function k, Function v)</code></td>
    <td rowspan="3">Crée une map en utilisant les fonctions de correspondance pour les clés, les valeurs, une fonction de fusion (merge function) et un type optionnel</td>
    <td rowspan="3"><code>Map</code></td>
  </tr>
  <tr>
    <td><code>toMap(Function k, Function v, BinaryOperator m)</code></td>
  </tr>
  <tr>
    <td><code>toMap(Function k, Function v, BinaryOperator m, Supplier s)</code></td>
  </tr>
</tbody>
</table>

#### Collecter avec les collecteurs basiques
La plupart des collecteurs marchent de la même manière.
```java
// Exemple d'utilisation de collecteur -- basique
Stream<String> ohMy = Stream.of("lions", "tigers", "bears");
String result = ohMy.collect(Collectors.joining(", "));
System.out.println(result); // lions, tigers, bears
```
Remarquez que les collecteurs sont dans la classe `Collectors` et non pas `Collector`, c'est un principe récurrent, vous l'avez déjà vu avec `Collections` et `Collection`
```java
// Exemple d'utilisation de collecteur -- avec fonction en  paramètre
Stream<String> ohMy = Stream.of("lions", "tigers", "bears");
String result = ohMy.collect(Collectors.averagingInt(String::length));
System.out.println(result); // 5.33333333333333333
```
Cette fois-ci on utilise une référence de méthode pour savoir sur quoi se baser pour faire la moyenne.

Avec du code écrit avant Java 8, on peut vouloir renvoyer des `Collection` plutôt que des `Stream`. Aucun problème, on peut faire nos opérations sur un `Stream` et convertir en `Collection` à la fin :
```java
// Exemple d'utilisation de collecteur -- transformer en Collection
Stream<String> ohMy = Stream.of("lions", "tigers", "bears");
TreeSet<String> result = ohMy.filter(s -> s.startsWith("t")).collect(Collectors.toCollection(TreeSet::new));
System.out.println(result); // [tigers]
```
#### Collecter vers des maps
```java
// Exemple de création de map à partir d'un stream
Stream<String> ohMy = Stream.of("lions", "tigers", "bears");
Map<String, Integer> map = ohMy.collect(
    Collectors.toMap(s -> s, String::length)
);
System.out.println(result); // {lions=5, bears=5, tigers=6}
```
Lorsque vous créez des maps, vous devez spécifier deux fonctions, la première détermine comment générer la clé (dans notre exemple `s -> s`), la seconde détermine comment générer la valeur (dans notre exemple `String::length`).

Retourner la même valeur passée à la lambda est très fréquent, Java fournit une méthode pour ça : `Function.identity()`

##### Piège
Maintenant nous voulons renverser la map et faire correspondre la taille du nom de l'animal avec l'animal lui-même :
```java
// Exemple de création de map à partir d'un stream -- problématique
Stream<String> ohMy = Stream.of("lions", "tigers", "bears");
Map<Integer, String> map = ohMy.collect(
    Collectors.toMap(String::length, k -> k) // BAD -- lance une exception
); 
```
lance une exception `Exception in thread "main" java.lang.IllegalStateException: Duplicate key lions`.

Le problème est que deux de ces noms d'animaux ont la même taille (donc la même clé). Or nous n'avons pas dit à Java quoi faire dans ce cas-là il nous lance donc une exception pour nous le faire savoir.
```java
// Exemple de création de map à partir d'un stream -- solution
Stream<String> ohMy = Stream.of("lions", "tigers", "bears");
Map<Integer, String> map = ohMy.collect(
    Collectors.toMap(String::length, k -> k, (s1, s2) -> s1 + "," + s2) // GOOD -- on a défini une stratégie
); 
System.out.println(map); // {5=lions,bears, 6=tigers}
System.out.println(map.getClass()); // class. java.util.HashMap -- AUCUNE GARANTIE -- peut être n'importe quelle implémentation de map
```
Si nous voulons une implémentation particulière de `Map` nous pouvons :
```java
// Exemple de création de map à partir d'un stream -- solution
Stream<String> ohMy = Stream.of("lions", "tigers", "bears");
Map<Integer, String> map = ohMy.collect(
    Collectors.toMap(String::length, k -> k, (s1, s2) -> s1 + "," + s2, TreeMap::new)
); 
System.out.println(map); // {5=lions,bears, 6=tigers}
System.out.println(map.getClass()); // class. java.util.TreeMap -- GARANTI
```
#### Collecter en utilisant `grouping`, `partitioning` et `mapping`
##### `groupingBy`
Supposons que nous voulions avoir des groupes de noms par leur taille :
```java
Stream<String> ohMy = Stream.of("lions", "tigers", "bears");
Map<Integer, List<String>> map = ohMy.collect(
    Collectors.groupingBy(String::length)
); 
System.out.println(map); // {5=[lions,bears], 6=[tigers]}
```
Maintenant supposons que nous voulions un `Set` à la place des `List` :
```java
Stream<String> ohMy = Stream.of("lions", "tigers", "bears");
Map<Integer, Set<String>> map = ohMy.collect(
    Collectors.groupingBy(String::length, Collectors.toSet())
); 
System.out.println(map); // {5=[lions,bears], 6=[tigers]}
```

On peut même changer le type de map renvoyé : 
```java
Stream<String> ohMy = Stream.of("lions", "tigers", "bears");
TreeMap<Integer, Set<String>> map = ohMy.collect(
    Collectors.groupingBy(String::length, TreeMap::new, Collectors.toSet())
); 
System.out.println(map); // {5=[lions,bears], 6=[tigers]}
```
C'est très flexible.
##### `partitioningBy()`
*partitioning* est un cas particulier de *grouping*.

Avec *partitioning* il n'y a que deux groupes possibles : `true` et `false`. C'est comme diviser une liste en deux parties
```java
// Exemple d'utilisation de partitioningBy
Stream<String> ohMy = Stream.of("lions", "tigers", "bears");
Map<Boolean, List<String>> map = ohMy.collect(
    Collectors.partitioningBy(s -> s.length() <= 5)
); 
System.out.println(map); // {false=[tigers], true=[lions, bears]}
```
```java
// Exemple d'utilisation de partitioningBy -- liste vide
Stream<String> ohMy = Stream.of("lions", "tigers", "bears");
Map<Boolean, List<String>> map = ohMy.collect(
    Collectors.partitioningBy(s -> s.length() <= 7)
); 
System.out.println(map); // {false=[], true=[lions, tigers, bears]}
```
Remarquez que la liste existe toujours même si elle est vide.

Comme avec `groupingBy()`, on peut changer le type de `List` vers quelque chose d'autre.
```java
// Exemple d'utilisation de partitioningBy -- Changement de type de clé
Stream<String> ohMy = Stream.of("lions", "tigers", "bears");
Map<Boolean, Set<String>> map = ohMy.collect(
    Collectors.partitioningBy(s -> s.length() <= 7, Collectors.toSet())
); 
System.out.println(map); // {false=[], true=[lions, tigers, bears]}
```

Contrairement à `groupingBy()` vous ne pouvez pas changer le type de `Map` retourné.

Au lieu d'utiliser le collecteur (*downstream collector*) pour définir le type on peut utiliser n'importe quel collecteur vu précédemment.
```java
// Exemple d'utilisation de partitioningBy -- collecteur
Stream<String> ohMy = Stream.of("lions", "tigers", "bears");
Map<Boolean, Long> map = ohMy.collect(
    Collectors.partitioningBy(String::length, Collectors.counting())
); 
System.out.println(map); // {5=2, 6=1}
```
##### `mapping`
Le collecteur mapping nous aide à descendre d'un niveau et ajouter un autre collecteur.
```java
// Exemple d'utilisation de mapping()
Stream<String> ohMy = Stream.of("lions", "tigers", "bears");
Map<Integer, Optional<Character>> map = ohMy.collect(
    Collectors.groupingBy(
        String::length,
        Collectors.mapping(
            s -> s.charAt(0),
            Collectors.minBy(Comparator.naturalOrder()) 
        )
    )
);
System.out.println(map); // {5=Optional[b], 6=Optional[t]}
```
Mapping prend deux paramètres, le premier est la fonction pour générer la valeur et le second pour le regrouper d'avantage.

Vous verrez souvent `Collectors` utilisé avec un import statique pour réduire la taille du code.
```java
// Exemple d'utilisation de mapping() -- import statique 
import static java.util.stream.Collectors.*;
// ou
import static java.util.stream.Collectors;
// [...]
Stream<String> ohMy = Stream.of("lions", "tigers", "bears");
Map<Integer, Optional<Character>> map = ohMy.collect(
    groupingBy(
        String::length,
        mapping(
            s -> s.charAt(0),
            minBy(Comparator.naturalOrder()) 
        )
    )
);
System.out.println(map); // {5=Optional[b], 6=Optional[t]}
```

Il existe un autre collecteur appelé `reducing()`, vous n'avez pas besoin de le connaître pour l'examen.

C'est un cas général de réduction dans le cas ou tous les autres collecteurs ne répondent pas à vos besoins.

### Débugger des génériques complexes
Lorsque vous travaillez avec `collect()`, il y a souvent plusieurs niveaux de génériques qui rendent les erreurs de compilation illisible.

Voici plusieurs techniques pour faire face à cette situation :
- Recommencez avec une instruction simple et ajoutez-en petit à petit pour savoir quelle partie du code introduit l'erreur,
- Extrayez les parties de l'instruction en instruction séparées, si elles compilent vous saurez que le problème est autre part,
- Utilisez *des wildcards génériques* pour le type de retour final, si cela compile c'est que le type de retour n'est pas celui que vous attendiez.