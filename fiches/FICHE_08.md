## Utiliser les *Streams*
Un *Stream* en java est une séquence de données.

Une *stream pipeline* est uen série d'opérations qui opèrent sur un *stream* pour produire un résultat.

Une *stream pipeline* peut être comparée à une chaine de production dans la vraie vie.

Il y a trois parties dans une *stream pipeline* :
- **La source** : d'où vient le *stream*,
- **Les opérations intermédiaires** : qui transforment un *stream* en d'autres *streams*. Il peut y avoir quelques ou autant d'opérations intermédiaires que vous le souhaitez. Puisque les streams utilisent la *lazy evaluation*, les opérations intermédiaires ne sont pas appelées jusqu'à ce que l'opération terminale soit appelée.
- **L'opération terminale** : Qui produit le résultat. Puisque les *streams* ne peuvent être utilisés qu'une seule fois, le *stream* n'est plus valide après la completion de l'operation terminale.

```
       _______________________________
Source |                             | Résultat de l'operation terminale
------>|  Operations intermediaires  | ---------->
       |_____________________________|
```
Remarquez que les opérations intermédiaires sont une boîte noire. 

Lorsque vous regardez la chaîne de production de l'extérieur, vous ne vous souciez que de l'entrée et de la sortie. Ce qui se passe entre temps est un détail d'implémentation. 

> Vous aurez à connaître la différence entre les opérations intermédiaires et terminales.

|Scenario|Pour les opérations intermédiaires ?|Pour les opérations terminales ?|
|---|:---:|:---:|
|Partie obligatoire d'une *pipeline* utile ?|❌|✔|
|Peut exister plusieurs fois dans une *pipeline* ?|✔|❌|
|Le type de retour est de type *stream* ?|✔|❌|
|Exécuté à l'appel de la méthode ?|❌|✔|
|*Stream* valide après l'appel ?|✔|❌|

### Créer des sources de *stream*
En Java, l'interface `Stream` se trouve dans le package `java.util.stream`.

Il y a plusieurs façons de créer un *stream fini* :
```java
Stream<String> empty = Stream.empty();          // count = 0 - Création d'un stream vide
Stream<Integer> singleElement = Stream.of(1);   // count = 1 - Création d'un stream à  partir d'un seul élément
Stream<String> fromArray = Stream.of(1, 2, 3);  // count = 2 - Création d'un stream à partir d'un array - la signature de la méthode prend des varargs en paramètre
```

Java fournit des moyens pratiques pour convertir des listes en streams :
```java
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> fromList = list.stream();                    // Création d'un stream à partir d'une liste
Stream<String> fromListParallel = list.parallelStream();    // Création d'un stream à partir d'une liste, permettant le traitement des éléments en parallèle
```
Création de *streams infinis* :
```java
Stream<Double> randoms = Stream.generate(Math::random);     // génère un stream infini de nombres aléatoires
Stream<Integer> oddNumbers = Stream.iterate(1, n -> n + 2); // génère un stream de nombres impaires
```
Par exemple, si vous appelez `randoms.forEach(System.out::println)` le programme ne s'arrêtera pas jusqu'à ce que vous le tuiez.

Plus tard dans ce chapitre vous verrez des opérations comme `limit()` pour transformer un *stream infini* en *stream fini*.

`iterate()` prend une valeur souche (*seed*) comme valeur de départ. Cet élément fera partie du *stream*. Le deuxième paramètre est une *expression lambda* qui est appliquée à la valeur précédente pour générer la valeur suivante.

> Si vous essayez de faire `System.out.println(stream)` vous obtiendrez quelque chose comme `java.util.stream.ReferencePipeline$3@4517d9a3`.
>C'est différent d'un `Collection` où vous pouvez voir le contenu.

### Utiliser les principales opérations terminales
Vous pouvez utiliser une *opération terminale* sans *opération intermédiaire* **mais pas l'inverse**.

Les *réductions* sont un type spécial d'*opérations terminales* qui prend le contenu du stream et le combine en une seule primitive ou `Object`.

|Méthode|Que se passe-t-il pour les *streams infinis*|Valeur retournée|Réduction|
|---|:---:|:---:|:---:|
|`allMatch()` / `anyMatch()` / `noneMatch()`|S'interrompt parfois|`boolean`|❌|
|`collect()`|Ne s'interrompt pas|Varie|✔|
|`count()`|Ne s'interrompt pas|`long`|✔|
|`findAny()` / `findFirst()`|S'interrompt|`Optional<T>`|❌|
|`forEach()`|Ne s'interrompt pas|`void`|❌|
|`min()` / `max()`|Ne s'interrompt pas|`Optional<T>`|✔|
|`reduce()`|Ne s'interrompt pas|Varie|✔|

#### `count()`
La méthode `count()` détermine le nombre d'éléments dans un stream fini.

Dans un stream infini, elle compte à l'infini (à éviter).

`count()` est une réduction car elle recherche chaque élément dans le *stream* et retourne une seule valeur.

La signature de la méthode est :
```java
long count()
```

```java
// Exemple d'utilisation de count()
Stream<String> s = Stream.of("monkey", "gorilla", "bonobo");
System.out.println(s.count()); // 3
```

#### `min()` et `max()`
Les méthodes `min()` et `max()` vous permettent de passer un comparateur personnalisé et de trouver la plus petite ou la plus grande valeur dans un stream fini selon cet ordre de tri.

Comme pour `count()`, pour un stream infini elles recherchent à l'infini car elles ne peuvent pas savoir si une valeur plus grande ou plus petite est à venir.

Ces deux méthodes sont des réductions car elles retournent une unique valeur après avoir cherché dans tout le stream.

Les signatures de ces méthodes sont :
```
Optional<T> min(<? super T> comparator)
Optional<T> max(<? super T> comparator)
```

```java
// Exemple d'utilisation de min() 
Stream<String> s = Stream.of("monkey", "ape", "bonobo");
Optional<String> min = s.min((s1, s2) -> s1.length() - s2.length());
min.ifPresent(System.out::println); // ape
```

```java
// Exemple s'il n'y a pas de minimum
Optional<T> minEmpty = Stream.empty().min((s1, s2) -> 0);
System.out.println(minEmpty.usPresent()); // false
```
Puisque le stream est vide le comparateur n'est jamais appelé et aucune valeur n'est présente dans l'`Optional`.
#### `findAny()` et `findFirst()`
Les méthodes `findAny()` et `findFirst()` retournent un élément du *stream* à moins que le *stream* soit vide.

Si le *stream* est vide, elles renvoient un `Optional` vide.

Ces méthodes marchent avec un *stream infini*. Puisque Java ne génère uniquement la quantité de *stream* nécessaire, le stream infini n'est obligé que de générer un seul élément.

`findAny()` est pratique lorsque vous travaillez avec des *streams* parallèles. Cela donne à java la flexibilité de vous retourner le premier élément qui vient plutôt que celui qui à besoin d'être premiers sur la base des opérations intermédiaires.

Ces méthodes sont des opérations terminales mais **ne sont pas** des réductions car elles retournent sans prendre en compte tous les éléments du *stream*.

Cela veut dire qu'elles retournent une valeur basée sur le *stream* mais ne réduisent pas le *stream* en une seule valeur.

Les signatures de ces méthodes sont :
```java
Optional<T> findAny()
Optional<T> findFirst()
```
```java
Stream<String> s = Stream.of("monkey", "gorilla", "bonobo");
Stream<String> infinite = Stream.generate(() -> "chimp");

s.findAny().ifPresent(System.out::println); // monkey
infinite.findAny().ifPresent(System.out::println); // chimp
```
Trouver une seule correspondance est plus utile que cela peut paraître. Des fois nous voulons juste prendre un échantillon de résultat et avoir un élément représentatif, plutôt que de traiter tous les éléments.
#### `allMatch()`, `anyMatch()` et `noneMatch()`
Les méthodes `allMatch()`, `anyMatch()` et `noneMatch()` cherchent dans un *stream* et retournent des informations sur ce *stream* concernant le *predicate*.

Ils peuvent ou ne peuvent pas s'interrompre sur des *streams infinis*. Cela dépend des données.

Comme la méthode `find()`, ce ne sont pas des réductions parce qu'elles ne prennent pas forcément en compte tous les éléments.

Les signatures de ces méthodes sont :
```java
boolean anyMatch(Predicate <? super T> predicate)
boolean allMatch(Predicate <? super T> predicate)
boolean noneMatch(Predicate <? super T> predicate)
```
```java
// Exemple - cherche si le nom de l'animal commence par une lettre
List<String> list = Arrays.asList("monkey", "2", "chimp");
Stream<String> infinite = Stream.generate(() -> "chimp");

Predicate<String> pred = x -> Character.isLetter(x.charAt(0));

System.out.println(list.stream().anyMatch(pred));  // true
System.out.println(list.stream().allMatch(pred));  // false
System.out.println(list.stream().noneMatch(pred)); // false
System.out.println(infinite.anyMatch(pred));       // false
```
Cela montre que vous pouvez réutiliser le même `Predicate`, mais vous avez besoin d'un nouveau *stream* à chaque fois.
```java
// Particularités des streams infinis
System.out.println(infinite.allMatch(pred));       // tourne jusqu'à ce qu'on tue le programme
System.out.println(infinite.noneMatch(pred));      // tourne jusqu'à ce qu'on tue le programme
```
> Souvenez vous que `allMatch()`, `anyMatch()` et `noneMatch()` retournent un `boolean`.
> Par contraste, les méthodes de recherche renvoient un `Optional` car elles retournent u élément du *stream*.

#### `forEach()`
C'est une alternative aux boucles.

Appeler `forEach()` sur un *stream* infini ne s'interrompt jamais.

Puisqu'il n'y a pas de valeur de retour, ce n'est pas une réduction.

La signature de méthode est :
```java
void forEach(Consumer<? super T> action)
```
C'est la seule opération terminale qui renvoie `void`.
```java
// Exemple d'utilisation de forEach
Stream<String> s = Stream.of("Monkey", "Gorilla", "Bonobo");
s.forEach(System.out::print); // MonkeyGorillaBonobo
```
> Vous pouvez appeler `forEach()` directement sur une `Collection` ou sur un *stream*

Vous ne pouvez pas utiliser une boucle for sur un *stream* :
```java
Stream s = Stream.of(1)
for (Integer i : s) {} // NE COMPILE PAS
```
Les *streams* ne peuvent pas utiliser la boucle `for` traditionnelle car ils n'implémentent pas l'interface `Iterable`.
#### `reduce()`
La méthode `reduce()` combine un stream en un seul objet.

Comme son nom l'indique, c'est une réduction.

Les signatures de la méthode sont les suivantes :
```java
T reduce(T  identity, BinaryOperator<T> accumulator)                                            // cf. reduce avec identité et accumulateur

Optional<T> reduce(BinaryOperator<T> accumulator)                                               // cf. reduce avec accumulateur seul

<U> U reduce (U identity, BiFunction<U, ? super T, U> accumulator, BinaryOperator<U> combiner)  // cf. reduce avec identité, accumulateur et combinateur
```

##### `reduce` avec identité et accumulateur
La façon la plus simple de faire une réduction est de commencer par une valeur initiale et de continuellement la fusionner avec la valeur suivante.
```java
// Exemple d'utilisation de reduce()
Stream<String> stream = Stream.of("w", "o", "l", "f");

String word = strem.reduce("", (s, c) -> s + c);

System.out.println(word); // wolf
```
On peut refactoriser en utilisant une référence de méthode :
```java
// Exemple
Stream<String> stream = Stream.of("w", "o", "l", "f");

String word = strem.reduce("", String::concat);

System.out.println(word); // wolf
```

##### `reduce` avec accumulateur seul
Dans la plupart des cas, l'identité n'est pas nécessaire, java nous permet de ne pas la spécifier.

Si vous ne spécifiez pas d'identité, un `Optional` est retourné car il pourrait ne pas y avoir de données

On distingue trois cas : 
- Le *stream* est vide, un `Optional` vide est retourné
- Si le *stream* ne contient qu'un seul élément, il est retourné (dans l'`Optional`)
- Si le *stream* contient plusieurs éléments, l'accumulateur est appliqué et les combine (... puis retourne le résultat dans l'`Optional`)

```java
// exemple des trois cas
BinaryOperator<Integer> op = (a, b) -> a * b;

Stream<Integer> empty = Stream.empty();
Stream<Integer> oneElement = Stream.of(3);
Stream<Integer> threeElements = Stream.of(3, 5, 6);

empty.reduce(op).ifPresent(System.out::print); // no output
oneElement.reduce(op).ifPresent(System.out::print); // 3
threeElements.reduce(op).ifPresent(System.out::print); // 90
```
###### `reduce` avec identité, accumulateur et combinateur
Permet à Java de créer des réductions intermédiaires et de les combiner à la fin.

C'est très utile lorsque l'on utilise des *parallel streams*

```java
BinaryOperator<Integer> op = (a, b) -> a * b;
Stream<Integer> stream = Stream.of(3, 5, 6);
System.out.println(stream.reduce(1, op, op));
```
####  `collect()`
La méthode `collect()` est un type spécial de réduction appelée *réduction mutable*.

Elle est plus efficiente que les réductions normales car elles utilisent le même objet mutable lors de l'accumulation.

Les objets mutables les plus communs incluent `StringBuilder` et `ArrayList`.

Elle nous permet de convertir des données d'un stream dans une autre forme.

Les signatures de `collect()` sont :
```java
<R> R collect(Supplier<R> supplier, BiConsumer<R, ? super T> accumulator, BiConsumer<R,R> combiner)     // cf signature spécifique

<R,A> R collect(Collector<? super T, A, R> collector)                                                   // cf signature avec collecteur
```
##### Signature spécifique
```java
// Exemple de signature spécifique de collect
Stream<String> stream = Stream.of("w", "o", "l", "f");
StringBuilder word = stream.collect(StringBuilder::new, StringBuilder::append, StringBuilder::append);
```
Le premier paramètre est un `Supplier` qui crée un objet qui conservera les résultats lors de la collecte des données.

Le second paramètre est un `BiConsumer`, qui prend deux paramètres et ne retourne rien. Il est responsable de l'ajout de nouveaux éléments dans la collecte de donnée.

Le dernier paramètre est un autre `BiConsumer`. Il est responsable de la fusion de deux collectes . Cela est utile lorsque l'on opère en parallèle.

```java
// Exemple avec un accumulateur et un combinateur différent
Stream<String> stream = Stream.of("w", "o", "l", "f");
TreeSet<String> set = stream.collect(TreeSet::new, TreeSet::add, TreeSet::addAll);
System.out.println(set); // [f, l, o, w]
```

Cette signature vous permet d'implémenter votre propre collecteur.
##### Signature avec collecteur
Java fournit une interface avec les collecteurs communs.

Cette approche permet de rendre le code plus lisible.

```java
// Exemple avec l'interface Collectors
Stream<String> stream = Stream.of("w", "o", "l", "f");
TreeSet<String> set = stream.collect(Collectors.toCollection(TreeSet::new);
System.out.println(set); // [f, l, o, w]
```

Si nous n'avons pas besoin d'avoir le *set* trié, nous pouvons encore raccourcir ce code :
```java
// Exemple avec l'interface Collectors -- non trié
Stream<String> stream = Stream.of("w", "o", "l", "f");
TreeSet<String> set = stream.collect(Collectors.toSet();
System.out.println(set); // [f, w, l, o]
```
Vous pourriez avoir un résultat différent en appelant `toSet` car il n'y a aucune garantie sur l'implémentation de `Set` qui va être appelée.

> L'examen s'attend à ce que vous connaissiez les collecteurs communs et de savoir comment définir votre propre collecteur en passant un *supplier*, accumulateur et combinateur.
> Plus tard dans ce chapitre nous verrons plusieurs `Collectors` utilisés pour grouper des données.

### Utiliser les opérations intermédiaires communes
Contrairement aux opérations terminales, les opérations intermédiaires gèrent les streams infinis en renvoyant un stream infini.
 
Comme les éléments ne sont produits que lorsqu'ils sont nécessaires cela ne pose pas de problème.
#### `filter()`
La méthode `filter()` retourne un `Stream` avec les éléments qui correspondent à l'expression.

la signature de méthode de `filter()` est :
```java
Stream<T> filter(Predicate<? super T> predicate)
```
Il est facile de se souvenir de cette opération et est très puissante car on peut lui passer n'importe quel `Predicate`
```java
// Exemple d'utilisation de filter()
Stream<String> s = Stream.of("monkey", "gorilla", bonobo);
s.filter(x -> x.startsWith("m")).forEach(System.out::print); // monkey
```
#### `distinct()`
La méthode `distinct()` retourne un stream avec ses valeurs dupliquées supprimées. 

Les doublons n'ont pas besoin d'être adjacents pour être supprimés.

Java appelle la méthode `equals()` pour déterminer si deux objets sont les mêmes.

La signature de la méthode `distinct()` est :
```java
Stream<T> distinct()
```
```java
// Exemple d'utilisation de distinct()
Stream<String> s = Stream.of("duck", "duck", "duck", "goose");
s.distinct().forEach(System.out::print) // duckgoose
```
#### `limit()` et `skip()`
Les méthodes `limit()` et `skip()` rendent un *stream* plus petit.

Elles peuvent rapetisser un stream fini, ou faire d'un stream infini un stream fini.

Les signatures de `limit()` et `skip()` sont :
```java
Stream<T> limit(int maxSize)
Stream<T> skip(int n)
```
```java
// Exemple d'utilisation des méthodes skip() et limit()
// saute les 5 premiers éléments et affiche les deux suivants
Stream<Integer> s = Stream.iterate(1, n -> n + 1);
s.skip(5).limit(2).forEach(System.out::print); // 67 
```
#### `map()`
La méthode `map()` crée une relation *one-to-one* entre les éléments du stream et les éléments de la prochaine étape du *stream*.

La signature de la méthode `map()` est :
```java
<R> Stream<R> map(Function<? super T, ? extends R> mapper)
```

Elle utilise une expression lambda pour trouver le type passé a la fonction et celui retourné.

> La méthode `map()` est utilisée pour transformer les données. Ne la confondez pas avec l'interface Map qui lie des clés et des valeurs.

```java
// Exemple d'utilisation de map()
Stream<String> s = Stream.of("monkey", "gorilla", "bonobo");
s.map(String::length).forEach(System.out::print); // 676
```
> Souvenez-vous que `String::length` est un raccourci pour l'expression lambda `x -> x.length()`, qui montre clairement qu'elle convertit un `String` en `Integer`.

#### `flatMap()`
La méthode `flatMap()` prend chaque élément du stream et fait en sorte que tous les éléments contenus soit au premier niveau dans un unique *stream*.

C'est pratique lorsque l'on cherche à supprimer les éléments vides ou lorsque vous souhaitezx combiner des streams de liste.

Vous n'êtes pas obligés de l'apprendre, mais la signature de `flatMap()` est : 
```java
<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper)
```
Il faut juste comprendre que la méthode retourne un Stream du type de la fonction au plus bas niveau.

```java
// Exemple d'utilisation de flatMap()
List<String> zero = Arrays.asList();
List<String> one = Arrays.asList("Bonobo");
List<String> two = Arrays.asList("Mama Gorilla", "Baby Gorilla");
Stream<List<String>>animals = Stram.of(zero, one, two);

animals.flatMap(l -> l.stream()).forEach(System.out.prinln);

/* Output :
 * Bonobo 
 * Mama Gorilla
 * Baby Gorilla
 */
```
#### `sorted()`
La méthode `sorted()` retourne un *stream* avec les éléments triés.

Comme pour les *arrays*, java utilise l'ordre naturel sauf si on lui spécifie un comparateur.

La signature de la méthode `sorted()` est :
```java
Stream<T> sorted()
Stream<T> sorted(Comparator<? super T> comparator)
```
Appeler la première signature utilise l'ordre de tri par défaut
```java
// Exemple d'utilisation de sorted() -- tri naturel
Stream<String> s = Stream.of("bear-", "brown-");
s.sorted().forEach(System.out::print); // bear-brown-
```
Souvenez-vous que l'on peut utiliser des expressions lambda comme comparateur
```java
// Exemple d'utilisation de sorted() -- comparateur
Stream<String> s = Stream.of("brown bear-", "grizzly-");
s.sorted(Comparator.reverseOrder()).forEach(System.out::print); // grizzly-brown bear-
```
###### Piège
```java
s.sorted(Comparator::reverseOrder); // NE COMPILE PAS
```
Comparator est une interface fonctionnelle, ce qui veut dire que l'on peut utiliser des références de méthode ou des expressions lambdas pour l'implémenter.

L'interface Comparator implémente une méthode qui prend deux paramètres String et retourne un int.

Or, `Comparator::reverseOrder` référence une fonction qui prend zéro paramètre et retourne un `Comparator`. Ce n'est pas compatible avec l'interface.

> Nous devons utiliser une méthode et non une référence de méthode.

#### `peek()`
La méthode `peek()` nous permet d'effectuer une opération sur le stream sans le modifier.

Elle est pratique pour débugger.

La signature de la méthode `peek()` est :
```java
Stream<T> peek(Consumer<? super T> action)
```
L'utilisation la plus fréquente de peek est pour afficher le contenu du stream et continuer normalement.

```java
// Exemple d'utilisation de peek()
Stream<String> stream = Stream.of("black bear", "brown bear", "grizzly");
long count = stream.filter(s -> s.startsWith("g")).peek(System.out::println).count(); // grizzly
System.out.println(count); // 1
```
> Lorsque l'on travaille avec des `Queue`, `peek()` ne regarde que le premier élément.
> Dans un stream, `peek()` regarde tous les éléments dans cette partie de la *stream pipeline*

> Danger : Changer d'état avec `peek()`
>
> `peek()` est prévu pour faire une opération sans changer le résultat.
> Or Java ne nous empêche pas de le faire ... 

### Mise en place d'une pipeline
#### Attention à l'ordre des opérations intermédiaires
```java
Stream.generate(() -> "Elsa")
    .filter(n -> n.length() == 4)
    .sorted() // TOURNE A L'INFINI car le stream est infini
    .limit(2)
    .forEach(System.out::println);
```
Alors que 
```java
Stream.generate(() -> "Elsa")
    .filter(n -> n.length() == 4)
    .limit(2)
    .sorted()
    .forEach(System.out::println); // ElsaElsa
```
#### `peek()` en coulisses
```java
// Exemple de base
Stream<Integer> infinite = Stream.iterate(1, x -> x + 1);
infinite.limit(5)
    .filter(x -> x % 2 == 1)
    .forEach(System.out::print); // 135
```

```java
// Exemple avec peek()
Stream<Integer> infinite = Stream.iterate(1, x -> x + 1);
infinite.limit(5)
    .peek(System.out::print)
    .filter(x -> x % 2 == 1)
    .forEach(System.out::print); // 11233455
```
```java
// Exemple avec ordre d'appel inversé 
Stream<Integer> infinite = Stream.iterate(1, x -> x + 1);
infinite.filter(x -> x % 2 == 1)
    .limit(5)
    .forEach(System.out::print); // 13579
```
```java
// Exemple avec ordre d'appel inversé et peek()
Stream<Integer> infinite = Stream.iterate(1, x -> x + 1);
infinite.filter(x -> x % 2 == 1)
    .peek(System.out::print)
    .limit(5)
    .forEach(System.out::print); // 1133557799
```
### Afficher un *stream*
Vous avez différentes manières d'afficher un stream puisque les opérations intermédiaires ne s'exécutent que si nécessaire.

|Option|Marche avec des streams infinis|Destructeur pour le stream|
|---|:---:|:---:|
|`s.forEach(System.out::println);`|❌|✔|
|`System.out.println(s.collect(Collectors.toList()));`|❌|✔|
|`s.peek(System.out::println).count();`|❌|❌|
|`s.limit(5).forEach(System.out::println);`|✔|✔|

Notez que la plupart de ces approches sont destructives, vous ne pourrez pas réutiliser le stream après l'affichage.

Une seule approche marche avec les streams infinis, Elle limite le nombre d'éléments avant l'affichage.

Si vous essayez les autres approches avec un stream infini, elles tourneront jusqu'à ce que vous tuiez le programme.