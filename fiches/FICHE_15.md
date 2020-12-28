## Travailler avec les streams parallèles

Jusqu'à maintenant tous les streams que nous avons vus sont des *streams serial*.

Un *stream serial* est un stream pour lequel les résultats sont ordonnés, avec une seule entrée étant traitée à la fois.

Un *stream parallèle* est un stream qui est capable de traiter les résultats de façon concurrente, en utilisant plusieurs threads.

Utiliser un stream parallèle peut à la fois changer les performances de votre application mais aussi changer les résultats.

Comme vous le verrez, certaines opérations requièrent une gestion particulière pour pouvoir être traitées avec un stream parallèle.

> Par défaut, le nombre de threads disponibles dans un stream parallèle est lié au nombre de CPUs disponibles dans votre environnement.
> Pour pouvoir augmenter le nombre de threads, vous aurez besoin de créer votre propre classe personnalisée.

### Créer un stream parallèle

L'API Streams a été conçue pour que la création de streams parallèles soit facile.

À l'examen vous devez connaître les deux façons de créer un stream parallèle.

#### `parallel()`

La première façon de créer un stream parallèle est avec un stream déjà existant.

Vous avez juste besoin d'appeler `parallel()` sur un stream existant pour créer un stream qui supporte un traitement *multi-threaded*.

```java
Stream<Integer> stream = Arrays.asList(1,2,3,4,5,6).stream();
Stream<Integer> parallelStream = stream.parallel();
```

Soyez conscients que `parallel()` est une opération intermédiaire qui change le stream originel.

#### `parallelStream()`

La seconde manière de créer un stream parallèle est avec la classe Java collection. L'interface `Collection` inclut une méthode `parallelStream()` qui peut être
appelée sur n'importe quelle collection et retourne un stream parallèle.

```java
Stream<Integer> parallelStream2 = Arrays.asList(1,2,3,4,5,6).parallelStream();
```

Nous utiliserons `parallelStream` sur des objets `Collection` pendant cette section.

> L'interface `Stream` inclut une méthode `isParallel()` qui peut être utilisée pour savoir si une instance de stream supporte le traitement parallèle.
> Certaines opérations sur les streams conservent l'attribut *parallel* alors que d'autres non.
> Par exemple, `Stream.concat(Stream s1, Stream s2)` sera parallèle si `s1` ou `s2` est parallèle.
> D'autre part, `flatMap()` créera un nouveau stream non-parallèle par défaut, sans tenir compte du fait que l'élément sous-jacent soit parallèle.

### Traiter les tâches en parallèle

#### Exemple avec un stream serial

```java
Arrays.asList(1,2,3,4,5,6)
    .stream()
    .forEach(s->System.out.println(s+" ");
```

```
1 2 3 4 5 6 
```

Comme vous pouviez vous y attendre, les résultats sont ordonnés et sont prévisibles

#### Exemple avec un stream parallèle

```java
Arrays.asList(1,2,3,4,5,6)
    .parallelStream()
    .forEach(s->System.out.println(s+" ");
```

```
4 1 6 5 2 3
5 2 1 3 6 4
1 2 6 4 5 3
```

Comme vous pouvez le voir, les résultats ne sont plus ordonnés ni ne sont prévisibles.

L'opération `forEach()` sur un stream parallèle est équivalent à soumettre plusieurs expression lambda `Runnable` à un *thread executor pooled*.

### Ordonner les résultats de `forEach`

L'API Streams inclut une version alternative de `forEach()`, appelée `forEachOrdered()`, qui force un stream parallèle à traiter les résultats dans l'ordre au
prix de performances.

```java
Arrays.asList(1,2,3,4,5,6)
    .parallelStream()
    .forEachOrdered(s->System.out.println(s+" ");
```

```
1 2 3 4 5 6 
```

Pourquoi utiliser cette méthode ? Dans votre application, vous pourriez avoir besoin d'appeler cette méthode dans une section qui prendrait des streams serial
et des streams parallèles, et vous auriez besoin de vous assurer que les résultats sont traités dans un ordre particulier.

Les méthodes appelées avant / après `forEachOrdered()` profiteraient quand même de l'amélioration de performance des streams parallèles.

### Comprendre les améliorations de performance

#### Exemple avec un stream serial

```java
import java.util.*;

public class WhaleDataCalculator {

    public int processRecord(int input) {
        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            // Gestion de l'interruptedException
        }
        return input + 1;
    }

    public void processAllData(List<Integer> data) {
        data.stream()
            .map(a -> processRecord(a))
            .count();
    }

    public static void main(String[] args) {
        WhaleDataCalculator calculator = new WhaleDataCalculator();

        // Définition des données
        List<Integer> data = new ArrayList<Integer>();
        for (int i = 0; i < 4000; i++) data.add(i);

        // Traitement des données
        long start = System.currentTimeMillis();
        calculator.processAllData(data);
        double time = (System.currentTimeMillis - start) / 1000.0;

        // Rapport des résultats
        System.out.println("Tâche complétée en " + time + " secondes");
    }
}
```

```
Tâche complétée en 40.044 secondes
```

#### Exemple avec un stream parallèle

```java
public void processAllData(List<Integer> data){
    data.parallelStream().map(a->processRecord(a)).count();
}
```

```
Tâche complétée en 10.542 secondes
```

Comme vous pouvez le voir, l'utilisation de streams parallèles peut entrainer une amelioration des résultats.

Encore mieux, les résultats *scale* avec le nombre de processeurs.

Le *scaling* est la propriété que plus on augmente le nombre de processeurs, plus les résultats s'améliorent.

Est-ce que tous nos streams doivent être parallèles ? Pas vraiment. L'utilisation de streams parallèles ont tendance à atteindre de meilleurs résultats lorsque
le nombre d'éléments du stream est suffisamment large.

Pour les petits streams, l'amélioration est souvent limitée, et il existe des coûts indirects à l'allocation et à la mise en place de traitements parallèles.

> Il n'y a aucune garantie que l'utilisation de streams parallèles améliore les performances

### Comprendre les opérations indépendantes

Les streams parallèles peuvent améliorer les performances car ils s'appuient sur la propriété que plusieurs opérations de streams peuvent être exécutées
indépendamment.

Les opérations indépendantes sont les opérations pour lesquelles le résultat de cette opération sur un élément du stream n'a pas d'impact sur les résultats des
autres éléments du stream.

#### Exemple d'opération indépendante

```java
Arrays.asList("jackal","kangaroo","lemur")
    .parallelStream()
    .map(s->s.toUpperCase())
    .forEach(System.out::println);
```

Par exemple, le mapping de jackal -> JACKAL peut être exécuté indépendamment du mapping de kangaroo -> KANGAROO.

L'ordre ne peut pas être garanti.

```
KANGAROO
JACKAL
LEMUR
```

#### Exemple de mauvaise utilisation d'opération indépendante

```java
Arrays.asList("jackal","kangaroo","lemur")
    .parallelStream()
    .map(s->{System.out.println(s);return s.toUpperCase();})
    .forEach(System.out::println);
```

Il se pourrait que des résultats finaux soient affichés avant des résultats intermédiaires.

```
kangaroo
KANGAROO
lemur
jackal
JACKAL
LEMUR
```

Lorsque vous utilisez les streams, vous devez éviter d'utiliser des expressions lambdas qui peuvent entraîner des *side effects*.

> Pour l'examen, vous devez retenir que les streams parallèles peuvent traiter les résultats indépendamment, bien que l'ordre des résultats ne puisse être déterminé à l'avance.

### Éviter les opérations *stateful*

Des *side effects* peuvent aussi apparaître dans vos streams parallèles si vos expressions lambdas sont *stateful*.

Une expression lambda *stateful* est une expression dont le résultat dépend de tout état susceptible de changer pendant l'exécution du pipeline.

D'autre part, une expression lambda *stateless* est une expression dont le résultat ne dépend d'aucun état qui pourrait changer pendant l'exécution d'un
pipeline.

```java
List<Integer> data = Collections.synchronizedList(new ArrayList<>());
Arrays.asList(1,2,3,4,5,6).parallelStream()
    .map(i->{data.add(i);return i;}) // Évitez les expressions lambda stateful
    .forEachOrdered(i->System.out.print(i+" "));

System.out.println();
for(Integer e:data){
    System.out.print(e+" ");
}
```

```
1 2 3 4 5 6 
1 2 4 6 5 3 
```

La méthode `forEachOrder()` affiche les nombres du stream séquentiellement, alors que l'ordre dans lequel les éléments sont ajoutés dans la liste est aléatoire.

Vous pouvez voir que les expressions lambdas *stateful* qui modifient la liste de données en parallèle produit des résultats imprévisibles à l'exécution.

Il est fortement encouragé que vous évitiez les opérations stateful lorsque vous utilisez des streams parallèles pour éviter de potentiels *side effects*.

En pratique, elles devraient aussi être évitées avec les streams serial si possible, puisqu'elles empêcheraient vos streams de profiter des avantages de la
parallélisation.

#### Utiliser les collections concurrentes avec les streams parallèles

À chaque fois que vous travaillez avec une collection dans des streams parallèles, il est recommandé d'utiliser une collection concurrente.

Si nous avions utilisé une `ArrayList` non concurrente dans l'exemple précédent, nous aurions pu avoir comme résultat :

```
1 2 3 4 5 6 
null 2 4 5 6 1 
```

### Traitement des réductions parallèles

Les opérations de réduction sur des streams parallèles sont appelées *réductions parallèles*.

Les résultats de réductions parallèles peuvent être différents de ceux attendus lorsque vous utilisez des streams *serial*.

#### Traitement de tâches basées sur l'ordre

Puisque l'ordre n'est pas garanti avec les streams parallèles, les méthodes telles que `findAny()` sur les streams parallèles peuvent résulter en des
comportements inattendus.

##### Exemple avec un stream *serial*

```java
System.out.println(Arrays.asList(1,2,3,4,5,6).stream().findAny().get());
```

Ce code affiche systématiquement la première valeur du stream.

```
1
```

##### Exemple avec un stream serial

```java
System.out.println(Arrays.asList(1,2,3,4,5,6).parallelStream().skip(5).get());
```

Le résultat ici peut être 4, 1, ou n'importe quelle valeur du stream.

Vous pouvez donc voir qu'avec les streams parallèles, les résultats de `findAny()` ne sont plus prévisibles.

Toute opération basée sur l'ordre, incluant `findFist()`, `limit()`, ou `skip()`, peuvent s'effectuer plus lentement dans un environnement parallèle.

Le côté positif est que le résultat des opérations basées sur l'ordre sur un stream parallèle seront régulières avec un stream *serial*.

Par exemple, appeler `skip(5).limit(2).findFirst()` retournera le même résultat avec un stream *serial* ordonné qu'avec un stream parallèle.

#### Créer des streams non ordonnés

Tous les streams que nos avons vus sont considérés comme ordonnés par défaut, il est possible de créer un stream non ordonné depuis un stream ordonné, d'une
manière similaire à celle de la création d'un stream parallèle depuis un stream *serial*

```java
Arrays.asList(1,2,3,4,5,6).stream.unordered();
```

Cette méthode ne réordonne pas les éléments, elle notifie seulement à la JVM que si une méthode *order-based* est appliquée sur ce stream, l'ordre peut être
ignoré.

Par exemple, appeller `skip(5)` sur un stream non ordonné sautera n'importe quels 5 éléments, plutôt que les 5 premiers sur un stream ordonné.

Pour un stream *serial*, utiliser une version non ordonnée n'a pas d'effets, mais avec des streams parallèles, les résultats peuvent gagner en performances.

```java
Arrays.asList(1,2,3,4,5,6).stream.unordered().parallel();
```

Même si les streams non ordonnés ne sont pas à l'examen, si vous développez des applications avec des streams parallèles, vous devez savoir quand appliquer un
stream non ordonné pour améliorer les performances.

#### Combiner les résultats avec reduce()

L'opération `reduce()` combine un stream en un seul objet.

Le premier paramètre de la méthode `reduce()` est appelé *identité*, le deuxième est *l'accumulateur*, et le troisième est le *combinateur*.

```java
// Exemple d'utilisation de reduce()
System.out.println(Arrays.asList('w','o','l','f')
    .stream()
    .reduce("",(c,s1)->c+s1,(s2,s3)->s2+s3)
);
```

> Les noms utilisés dans cet exemple ne sont pas accidentels. `c` est une valeur `char` alors que `s1`, `s2` et `s3` sont des valeurs `String`.
> Souvenez-vous que dans la version avec 3 arguments de reduce, l'accumulateur est une `BiFunction` alors que le combinateur est un `BinaryOperator`.

Sur des streams parallèles, la méthode `reduce()` applique les réductions sur des paires d'éléments du stream pour créer des valeurs intermédiaires et les
combine pour produire le résultat final.

Contrairement aux streams *serial* qui à combiné `wolf` avec un caractère à la fois, les streams parallèles les chaîne intermédiaires `wo` et `lf` ont été
créées avant d'être combinées.

Avec les streams parallèles, nous pourrions avoir un problème d'ordre (`wlfo` ou `flwo`), mais l'API Streams évite ce problème en ne permettant aux streams
d'être traités en parallèle uniquement si les arguments de la méthode `reduce()` répondent à certains principes.

##### Critères pour les arguments de la méthode `reduce()`

- *L'identité* doit être définie telle que pour tout élément `u` du stream, `combiner.apply(identity, u)` est égal à `u`,
- L'opérateur *accumulateur* `op` doit être **associatif** et **stateless** tel que `(a op b) op c` soit égal à `a op (b op c)`,
- L'opérateur *combinateur* doit aussi être **associatif** et **stateless** et doit être **compatible** avec l'identité, tel que pou tout `u` et `t`
  , `combiner.apply(u, accumulator.apply(identity, t))` soit égal à `accumulator.apply(u, t)`.

Si vous suivez ces principes lorsque vous créez vos arguments de `reduce()`, alors les opérations peuvent être traitées en utilisant un stream parallèle et les
résultats seront ordonnés comme si vous utilisiez un stream *serial*.

Notez aussi que ces principes s'appliquent aussi à l'identité et à l'accumulateur lorsque vous utilisez la version de `reduce()` avec un ou deux arguments sur
un stream parallèle.

> Bien que les critères pour les arguments d'entré de `reduce()` soient vrais pour les streams *serial* et parallèles, vous n'avez jamais rencontré de problèmes avec les streams *serial* parce que les résultats étaient toujours ordonné.
> Pour les streams parallèles, l'ordre n'est pas garanti, et une seule violation de ces critères peut mener à des *side effects* ou à des résultats imprévisibles.

##### Exemple avec un accumulateur non associatif

```java
System.out.println(Arrays.asList(1,2,3,4,5,6)
    .parallelStream()
    .reduce(0,(a,b)->a-b))); // N'est PAS un accumulateur associatif 
```

Les résultats possibles sont -21, 3 ou n'importe quel autre valeur car l'accumulateur n'est pas associatif

##### Exemple avec une identité qui n'est pas une vraie valeur d'identité

```java
System.out.println(Arrays.asList('w','o','l','f')
    .parallelStream()
    .reduce("X",String::concat)); // N'est PAS un accumulateur associatif 
```

```
XwXoXlXf
```

##### Utilisez la méthode `reduce()` avec 3 arguments !

Bien que les versions de `reduce()` avec un et deux arguments supportent le traitement parallèle, il est recommandé d'utiliser la version avec 3 paramètres
lorsque vous travaillez avec des streams parallèles.

Fournir un *combinateur* explicite permet à la JVM de partitionner les opérations du stream plus efficacement.

#### Combiner les résultats avec `collect()`

Comme `reduce()`, l'API Streams fournit une version avec trois arguments de la méthode `collect()` qui prend en argument un opérateur *accumulateur*, un
opérateur *combinateur* et un opérateur *supplier* plutôt qu'une *identité*.

Comme `reduce()`, *l'accumulateur* et le *combinateur* doivent être **associatifs** et **stateless**, avec le *combinateur* **compatible** avec l'opérateur *
accumulateur*.

De cette manière, la version avec trois arguments de la méthode `collect()` peut être utilisée lors d'une réduction parallèle.

```java
Stream<String> stream = Stream.of("w","o","l","f").parallel();
SortedSet<String> set = stream.collect(ConcurrentSkilListSet::new,Set::add,Set::addAll);
System.out.println(set); // [f, l, o, w]
```

Souvenez-vous quel es éléments dans un `ConcurrentSkipListSet` sont triés dans l'ordre naturel.

Vous devez utiliser une collection concurrente pour combiner les résultats, pour s'assurer que les résultats de threads concurrents lèvent
une `ConcurrentModificationException`

##### Utiliser la méthode `collect()` à un argument

Souvenez-vous que la version à un argument de la méthode `collect()` prend un *collecteur*

```java
Stream<String> stream = Stream.of("w","o","l","f").parallel();
Set<String> set = stream.collect(Collectors.toSet());
System.out.println(set); // [f, w, l, o]
```

Procéder à une réduction parallèle avec un *collecteur* requiert des réflexions additionnelles, par exemple, si la collections vers laquelle vous insérez les
données est un set de donnée ordonnée, come une `List`, alors les éléments dans la collection résultante doivent être dans le même ordre, indépendamment de
l'utilisation d'un stream *serial* ou d'un stream parallèle.

Les règles suivantes assurent quel la réduction parallèle est effectuée de manière efficace en Java en utilisant un *collecteur*

###### Critères pour les arguments de la méthode `collect()`

- Le stream est **parallèle**,
- Le paramètre de l'opération de collection a la caractéristique `Collector.Characteristics.CONCURRENT`,
- Si le stream est non ordonné, alors le collecteur doit avoir la caractéristique `Collector.Characteristics.UNORDERED`.

Toute classe qui implémente l'interface `Collector` inclut une méthode `characteristics()` qui retourne un set d'attributs disponibles pour le *collecteur*.

Bien que `Collectors.toSet()` aie la caractéristique `UNORDERED`, il n'a pas la caractéristique `CONCURRENT`; par conséquent il ne peut pas être utilisé pour
une réduction parallèle.

La classe `Collectors` inclut deux sets de méthodes pour récupérer des collecteurs `UNORDERED` et `CONCURRENT`, `Collectors.toConcurrentMap()`
et `Collectors.groupingByConcurrent()` et permet donc de traiter des réductions parallèles efficacement.

###### # `toConcurrentMap()`

```java
Stream<String> ohMy = Stream.of("lions","tigers","bears").parallel();
ConcurrentMap<Integer, String> map = ohMy
    .collect(Collectors.toConcurrentMap(String::length,k->k,(s1,s2)-> s1 + "," + s2));

System.out.println(map); // {5=lions,bears, 6=tigers}
System.out.println(map.getClass()); // java.utili.concurrent.ConcurrentHashMap
```

Nous avons utilisé une référence `ConcurrentMap`, alors que la class retournée est `ConcurrentHashMap`. La classe spécifique n'est pas garantie, il s'agira
juste d'une classe qui implémente `ConcurrentMap`.

###### # `groupingByConcurrent()`

```java
Stream<String> ohMy = Stream.of("lions","tigers","bears").parallel();
ConcurrentMap<Integer, String> map = ohMy
    .collect(Collectors.groupingByConcurrent(String::length));

System.out.println(map); // {5=[lions,bears], 6=[tigers]}
```

Comme précédemment, le résultat peut être assigné à un référence de `ConcurrentMap`.

###### Encourager le traitement parallèle

Garantir qu'un stream effectuera une réduction en parallèle, plutôt que *single threaded*, est souvent difficile en pratique.

Par exemple, `reduce()` avec un argument sur un stream parallèle peut s'effectuer de façon concurrente même s'il n'a pas d'argument *combinateur* défini
explicitement. Ou bien vous pouvez vous attendre à ce que certains collecteurs performent mieux sur un stream parallèle alors qu'ils seront traités comme *
single threaded* à l'exécution.

La clé pour appliquer une réduction parallèle est d'encourager la JVM à prendre avantage des structures parallèles, par exemple, en utilisant le
collecteur `groupingByConcurrent` sur un stream parallèle plutôt que le collecteur `groupingBy`.

En encourageant la JVM à prendre avantage du traitement parallèle, nous avons les meilleures performances à l'exécution.

### Gérer les processus concurrents

L'API Concurrency inclut des classes qui peuvent être utilisées pour coordonner des tâches dans un groupe de threads associés.

Ces classes ont été conçues pour des scenarii spécifiques.

À l'examen vous devrez connaître `CyclicBarrier` et `ForkJoinPool`

#### Créer une `CyclicBarrier`

```java
// Exemple sans CyclicBarrier
import java.util.concurrent.*;

public class LionPenManager {

    private void removeAnimals() {
        System.out.println("Removing animals");
    }

    private void cleanPen() {
        System.out.println("Cleaning the pen");
    }

    private void addAnimals() {
        System.out.println("Adding animals");
    }

    public void performTasks() {
        removeAnimals();
        cleanPen();
        addAnimals();
    }

    public static void main(String[] args) {
        ExecutorService service = null;

        try {
            service = Executors.newFixedThreadPool(4);
            LionPenManager manager = new LionPenManager();
            for (int i = 0; i < 4; i++)
                service.submit(() -> manager.performTask());
        } finally {
            if (service != null) service.shutdown();
        }
    }
}
```

```
Removing animals
Removing animals
Cleaning the pen
Adding animals
Removing animals
Cleaning the pen
Adding animals
Removing animals
Cleaning the pen
Adding animals
Cleaning the pen
Adding animals
```

Bien qu'au sein d'un seul thread les résultats soient ordonnés, entre différents *workers* le résultat est aléatoire.

Nous pouvons améliorer ce résultat en utilisant la classe `CyclicBarrier`.

Le constructeur de `CyclicBarrier` prend en premier paramètre une valeur limite indiquant le nombre de threads à attendre.

À chaque fois qu'un thread se termine, la méthode `await()` est appelée. Une fois que tous les threads de la `CyclicBarrier` ont appelé la méthode `await()` la
« barrière est levée » et tous les threads peuvent continuer.

```java
// Exemple avec CyclicBarrier
import java.util.concurrent.*;

public class LionPenManager {

    private void removeAnimals() {
        System.out.println("Removing animals");
    }

    private void cleanPen() {
        System.out.println("Cleaning the pen");
    }

    private void addAnimals() {
        System.out.println("Adding animals");
    }

    public void performTasks(CyclicBarrier c1, CyclicBarrier c2) {
        try {
            removeAnimals();
            c1.await();
            cleanPen();
            c2.await();
            addAnimals();
        } catch (InturruptedException | BrokenBarrierException e) {
            // Gérer les exceptions checked ici
        }
    }

    public static void main(String[] args) {
        ExecutorService service = null;

        try {
            service = Executors.newFixedThreadPool(4);
            LionPenManager manager = new LionPenManager();
            CyclicBarrier c1 = new CyclicBarrier(4);
            CyclicBarrier c2 = new CyclicBarrier("*** Pen cleaned!");

            for (int i = 0; i < 4; i++)
                service.submit(() -> manager.performTask(c1, c2));
        } finally {
            if (service != null) service.shutdown();
        }
    }
}
```

```
Removing animals
Removing animals
Removing animals
Removing animals
Cleaning the pen
Cleaning the pen
Cleaning the pen
Cleaning the pen
*** Pen cleaned!
Adding animals
Adding animals
Adding animals
Adding animals
```

##### Taille du *thread pool* et limite de la *cyclic barrier*

Si vous utilisez un *thread pool*, soyez sûrs que vous avez défini un nombre de threads au moins aussi large que la valeur limite de la `CyclicBarrier`.

Si dans notre dernier exemple nous avions utilisé :

```java
ExecutorService service=Executors.newFixedThreadPool(2);
```

Le code aurait continué indéfiniment car la limite de la barrière ne serait jamais atteinte.

Comme vous le verrez plus tard, c'est une forme de *deadlock*.

La classe `CyclicBarrier` nous permet d'exécuter des tâches complexes et *multi-threaded*. Cette solution est meillerur que la solution *single-threaded* car
les tâches individuelles peuvent être exécutées en parallèle.

##### Réutiliser une `CyclicBarrier`

Après qu'une `CyclicBarrier` ait été brisée, tous les threads sont relâchés et le nombre de threads attendant dans la `CyclicBarrier` retombe à zéro.

À ce moment-là la `CyclicBarrier` peut être réutilisée pour un nouveau set de threads.

#### Appliquer le *framework Fork / Join*

##### Introduction à la récursion

Le *framework fork / join* s'appuie sur le concept de récursion pour résoudre des tâches complexes.

La récursion est le processus par lequel une tâche s'appelle elle-même pour résoudre un problème.

Une solution récursive est construite avec un cas de base et un cas récursif.

- *Cas de base* : Une méthode non récursive qui est utilisée pour arrêter le *path* récursif
- *Cas récursif* : Une méthode récursive qui peut s'appeler elle-même une ou plusieurs fois pour résoudre un problème.

```java
// Exemple de méthode récursive
public static int factorial(int n){
    if(n<=1) return 1;
    else return n*factorial(n-1);
}
```

Le plus gros challenge dans l'implémentation d'une solution récursive et d'être sûrs que le processus de récursion arrive au cas de base.

Si le cas de base n'est jamais atteint alors le programme ne s'arrêtera jamais. En Java, cela résulte par une `StackOverflowError` si une application se répète
trop de fois.

#### Règles d'application du framework fork / join

1. Création d'une `ForkJoinTask`,
2. Création d'un `ForkJoinPool`,
3. Démarrage de la `ForkJoinTask`.

La première tâche est la plus complexe car elle implique de définir le processus récursif.

Par chance, la deuxième et troisième tâche peuvent être complétés avec une seule ligne de code.

Pour l'examen, vous devrez savoir comment implémenter la *solution fork / join* en étendant l'une des deux classes `RecursiveAction` et `RecursiveTask` qui
implémentent toutes les deux l'interface `ForkJoinTask`.

La première classe, `RecursiveAction`, est une classe abstraite qui requiert d'implémenter la méthode `compute()`, qui retourne void, pour effectuer l'essentiel
du travail.

La seconde classe, `RecursiveTask`, est une classe abstraite générique qui requiert d'implémenter la méthode `compute()`, qui retourne un type générique, pour
effectuer l'essentiel du travail.

La différence entre `RecursiveAction` et `RecursiveTak` est similaire à la différence entre `Runnable` et `Callable` respectivement.

##### Exemple avec `RecursiveAction`

###### Création de la `RecursiveAction`

```java
import java.util.*;
import java.util.concurrent.*;

public class WeighAnimalAction extends RecursiveAction {

    private int start;
    private int end;
    private Double[] weight;

    public WeighAnimalAction(Double[] weights, int start, int end) {
        this.start = start;
        this.end = end;
        this.weight = weights;
    }

    protected void compute() {
        if (end - start <= 3) for (int i = start; i < end; i++) {
            weights[i] = (double) new Random().nextInt(100);
            System.out.println("Animal weighed" + i);
        }
        else {
            int middle = start + ((end - start) / 2);
            System.out, println("[start=" + start + ", middle=" + middle + ", end=" + end + "]");
            invokeAll(new WeightAnimalAction(weights, start, middle), new WeightAnimalAction(weights, middle, end));
        }
    }
}
```

###### Création de la `ForkJoinTask`

```java
public static void main(String[]args){
    Double[]weights = new Double[10];

    ForkJoinTask<?> task = new WeighAnimalAction(weights,0,weights.length);
    ForkJoinPool pool = new ForkJoinPool();
    pool.invoke(task);

    // Affichage des résultats
    System.out.println();
    System.out.print("Weights: ");
    Arrays.asList(weights).stream().forEach(d->System.out.print(d.intValue+" "));
}
```

Par défaut le `ForkJoinPool` utilisera le nombre de processeurs pour déterminer le nombre de threads à créer.

```
[start=0, middle=5, end=10]
[start=0, middle=2, end=5]
Animal Weighed: 0
Animal Weighed: 2
[start=5, middle=7, end=10]
Animal Weighed: 1
Animal Weighed: 3
Animal Weighed: 5
Animal Weighed: 6
Animal Weighed: 7
Animal Weighed: 8
Animal Weighed: 9
Animal Weighed: 4

Weights: 94 73 8 92 75 63 76 60 73 3
```

> Créer un `ForkJoinTask` et la soumettre à un `ForkJoinPool` ne garanti pas qu'elle soit exécutée immédiatement.
> Par exemple, une étape récursive peut générer 10 tâches lorsqu'il n'y a que 4 threads de disponibles, dans ce cas, comme un *pooled thread executor*, la tâche attendra un thread disponible pour traiter les données.

#### Travailler avec une `RecursiveTask`

```java
public class WeighAnimalAction extends RecursiveTask<Double> {

    private int start;
    private int end;
    private Double[] weight;

    public WeighAnimalAction(Double[] weights, int start, int end) {
        this.start = start;
        this.end = end;
        this.weight = weights;
    }

    protected void compute() {
        if (end - start <= 3) {
            for (int i = start; i < end; i++) {
                weights[i] = (double) new Random().nextInt(100);
                System.out.println("Animal weighed" + i);
                sum += weight[i];
            }
            return sum;
        } else {
            int middle = start + ((end - start) / 2);
            System.out, println("[start=" + start + ", middle=" + middle + ", end=" + end + "]");
            RecursiveTask<Double> otherTask = new WeighAnimalTask(weights, start, middle);
            otherTask.fork();                                                               // ✔ Exemple multi-threaded 
            return new WeighAnimalTask(weights, middle, end).compute() + otherTask.join();  // ✔ Exemple multi-threaded 
        }
    }
}
```

Puisque la méthode `invokeAll()` ne retourne pas de valeur, on la remplace par les commandes `fork()` et `join()` pour récupérer les données récursives.

La méthode `fork()` charge le *framework fork / join* de completer toutes les tâches dans des threads séparés.

La méthode `join()` fait attendre les résultats au thread courant.

On peut modifier la méthode main pour inclure toutes les données de la tâche :

```java
ForkJoinTask<?> task = new WeighAnimalAction(weights,0,weights.length);
ForkJoinPool pool = new ForkJoinPool();
Double sum = pool.invoke(task);

System.out.println("Sum : "+sum);
```

Avec notre exemple précédent, la somme totale aurait été `617`.

Faite attention à l'ordre dans lequel les méthodes `fork()` et `join()` sont appliquées.

La variation suivante est *single-threaded* :

```java
// ❌ Exemple single-threaded
ForkJoinTask<?> otherTask = new WeighAnimalAction(weights,weights.length);
Double otherResult = otherTask.fork().join();
return new WeighAnimalTask(weights,middle,end).compute() + otherTask.join();
```

Pour l'examen, soyez sûrs que `fork()` est appelé avant que le thread courant commence la sous-tâche et que `join()` soit appelé après que les résultats aient
été récupérés, pour que les résultats soient traités en parallèle.

#### Identifier les problèmes de fork / join

Contrairement aux autres classes et structures de l'API Concurrency, le *framework fork / join* peut être impressionnant pour les développeurs ne l'ayant pas vu
avant.

Voici une liste de conseils pour identifier les problèmes de classes *fork / join* à l'examen :

- La classe doit étendre `RecursiveAction` ou `RecursiveTask`,
- Si la classe étend `RecursiveAction`, alors elle doit surcharger une méthode protected `compute()` qui ne prend pas d'argument et retourne void,
- Si la classe étend `RecursiveTask`, alors elle doit surcharger une méthode protected `compute()` qui ne prend pas d'argument et retourne un type générique
  défini lors de la définition de la classe,
- La méthode `invokeAll()` prend deux instances de la classe *fork / join* et ne retourne pas de résultat,
- La méthode `fork()` fait qu'une nouvelle tâche est soumise au pool et est similaire à la méthode `submit` de *thread executor*,
- La méthode `join()` est appelée après la méthode `fork()` et fait que le thread courant attend le résultat d'une sous-tâche,
- Contrairement à `fork()`, appeler `compute()` dans la méthode fait que la tâche attend les résultats d'une sous-tâche,
- La méthode `fork()` doit être appelée avant que le thread courant fasse une opération `compute()`, avec `join()` appelé après pour lire les résultats,
- Puisque `compute()` ne prend pas d'argument, le constructeur de la classe est souvent utilisé pour passer des instructions à la tâche.

> Vous pouvez aussi utiliser `System.currentTimeMillis()` pour voir comment le *fork / join* améliore les performances.

## Identifier les problèmes de threads

Un problème de thread peut apparaître dans une application *multi-threaded* lorsque de threads ou plus interagissent de façon inattendue.

Par exemple, deux threads peuvent se bloquer l'accès a un segment de code particulier.

L'API Concurrency a été créée pour aider à éliminer les problèmes de threads potentiels communs aux développeurs.

Bien que l'API Concurrency réduit le potentiel de problèmes de threads, elle ne les élimine pas.

En pratique, trouver et identifier les problèmes de threads dans une application est souvent l'une des tâches les plus difficiles pour un développeur.

### Comprendre la *liveness* (Existence)

La *liveness* est la capacité d'une application à être exécutée sur le temps.

Les problèmes de *liveness* sont ceux quand l'application ne répond pas et est dans une sorte d'état « bloqué ».

Pour l'examen, il y a trois types de problèmes de *liveness* avec lesquels vous devez être habitués : **deadlock**, **starvation** et **livelock**.

#### Deadlock

Une *deadlock* apparaît lorsque deux threads ou plus sont bloqués pour toujours, l'un attendant l'autre et inversement.

```java
import java.util.concurrent.*;

public class Food {}

public class Water {}

public class Fox {

    public void eatAndDrink(Food food, Water water) {
        synchronized (food) {
            System.out.println("Got Food!");
            move();
            synchronized (water) {
                System.out.println("Got Water!");
            }
        }
    }

    public void drinkAndEat(Food food, Water water) {
        synchronized (water) {
            System.out.println("Got Water!");
            move();
            synchronized (food) {
                System.out.println("Got Food!");
            }
        }
    }

    public void move() {
        try {
            Thread.sleep();
        } catch (InterruptedException e) {
            // Gérer l'exception   
        }
    }

    public static void main(String[] args) {
        // Création des participants et des ressources
        Fox foxy = new Fox();
        Fox tails = new Fox();
        Food food = new Food();
        Water water = new Water();

        // Traitement des données
        ExecutorService service = null;
        try {
            service = Executors.newScheduledThreadPool(10);
            service.submit(() -> foxy.eatAndDrink(food, water));
            service.submit(() -> tails.drinkAndEat(food, water));
        } finally {
            if (service != null) service.shutdown();
        }
    }
}
```

```
Got Food!
Got Water!
// Attend indéfiniment
```

Cet exemple est considéré comme un *deadlock* car les deux participants sont bloqués de façon permanente, attendant des ressources qui seront jamais
disponibles.

##### Éviter les *deadlocks*

Comment résoudre une *deadlock* une fois qu'elle s'est produite ? Généralement vous ne pouvez pas.

D'un autre côté, il existe plusieurs stratégies pour vous aider à vous prémunir contre les *deadlocks*.

L'une des plus communes est que tous les threads ordonnent leurs demandes de ressources.

Il existe des techniques plus avancées pour détecter et résoudre les *deadlocks* en temps réel, mais sont difficiles à mettre en place et ont un succès limité.

De ce fait, la plupart des systèmes d'exploitations ignorent le problème et prétendent que les *deadlocks* n'arrivent jamais.

### *Starvation* (famine)

Une *starvation* arrive lorsque un thread se voit refuser répétitivement l'accès à une ressource partagée ou à un lock.

Le thread est toujours actif, mais est incapable d'effectuer son travail suite au fait que d'autres threads sont constamment en train de demander la ressource à
laquelle il essaie d'accéder.

#### Livelock

Un *livelock* arrive lorsque deux threads ou plus sont conceptuellement bloqués pour toujours, même s'ils sont tous les deux actifs et essayent de compléter
leurs tâches.

Le *livelock* est un cas particulier de *starvation* de ressource dans lequel deux threads ou plus essaient d'accéder à un set de *locks*, n'y arrivent pas et
recommencent une partie du processus.

En pratique, un *livelock* est difficile à détecter car les threads apparaissent comme actifs et peuvent répondre a des requêtes mais sont bloqués dans une
boucle infinie.

#### Gestion des *race conditions*

Une *race condition* est un résultat indésirable qui apparaît lorsque deux tâches, complétées séquentiellement, sont complétées au même moment.

Supposons la création d'un compte par deux utilisateurs avec le même username "ZooFan", se complétant en même temps. Les résultats possibles de cette *race
condition* sont :
- Les deux utilisateurs arrivent à créer un compte avec le nom ZooFan,
- Les deux utilisateurs ne peuvent pas créer de compte avec le nom ZooFan, résultant en un message d'erreur aux deux utilisateurs,
- Un utilisateur arrive à créer un compte avec le nom ZooName, pendant que l'autre reçoit un message d'erreur.

Pour l'examen, vous devez comprendre que les *race conditions* mènent à des données invalides si elles ne sont pas correctement gérées.

Même la solution ou les deux participants n'arrivent pas à créer de compte est préférable par rapport à laisse entrer des données invalides dans le système.


 