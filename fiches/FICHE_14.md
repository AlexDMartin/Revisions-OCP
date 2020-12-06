## Synchroniser l'accès aux données
La *thread-safety* est la propriété d'un objet qui garantie une exécution *safe* par plusieurs *threads* en même temps.

```java
import java.util.concurrent.*;

public class SheepManager {
    private int sheepCount = 0;

    private void incrementAndReport() {
        System.out.println((++sheepCount) + " ");
    }

    public static void main(String[] args) {
        ExecutorService service = null;

        try {
            service = Executors.newFixedThreadPool(20);

            SheepManager manager = new SheepManager()

            for(int i = 0; i < 10 ; i++)
                service.submit(() -> manager.incrementAndReport());            

        } finally {
            if(service != null) service.shutdown();
        }
    }
}
```
Souvenez-vous que l'opérateur pre-incremental ++ est un raccourci pour `sheepCount = sheepCount + 1` où la nouvelle valeur est retournée.

> Un problème apparaît lorsque nous exécutons la partie droite de cette expression

Lorsque deux threads essayent de lire et d'écrire les mêmes valeurs l'un des deux `++sheeplost` est perdu.

L'opérateur incrémental `++` n'est pas thread-safe.

Le résultat inattendu de deux tâches s'exécutant en même temps est appelé « *race condition* ».
```
// Exemples de résultat du programme
1 2 2 3 4 5 6 7 8 9 
2 4 5 6 7 8 1 9 10 3
2 1 3 4 5 6 7 8 9 10
```
### Protéger ses données avec les classes atomiques
Avec la sortie de l'API Concurrency, Java a ajouté un nouveau package `java.util.concurrent.atomic` pour nous aider a coordonner les accès à des valeurs primitives ou des références d'objet.

*Atomic* est la propriété d'une opération a être effectuée comme une seule unité d'exécution sans aucune interference avec d'autres threads.

Une version atomique thread-safe de l'opérateur incrément en serait une qui ferait la lecture et l'écriture de la variable comme une seule opération, et ne permettant aux autres threads d'accéder a la variable lors de cette opération.

Dans notre exemple, n'importe quel thread souhaitant accéder à la variable `sheepCount` lorsqu'une opération atomique est en cours devra attendre que l'opération atomique soit terminée.

#### Classes atomiques

|Nom de la classe|Description|
|---|---|
|`AtomicBoolean`|Une valeur `boolean` qui peut être mise à jour de façon atomique|
|`AtomicInteger`|Une valeur `int` qui peut être mise à jour de façon atomique|
|`AtomicIntegerArray`|Un tableau de `int` dans lequel les éléments peuvent être mis à jour de façon atomique|
|`AtomicLong`|Une valeur `long` qui peut être mise à jour de façon atomique|
|`AtomicLongArray`|Un tableau de `long` dans lequel les éléments peuvent être mis à jour de façon atomique|
|`AtomicReference`|Une référence d'objet générique qui peut être mise à jour de façon atomique|
|`AtomicReferenceArray`|Un tableau de références d'objets génériques dans lequel les éléments peuvent être mis à jour de façon atomique|

#### Méthodes atomiques
|Nom de la méthode|Description|
|---|---|
|`get()`|Retourne la valeur courante|
|`set()`|définit la valeur donnée, équivalant à l'opérateur d'assignation `=`|
|`getAndSet()`|définit la nouvelle valeur atomiquement et retourne l'ancienne valeur|
|`incrementAndGet()`|Pour les classes numériques, opération pré-incrémentale atomique équivalente a `++value`|
|`getAndIncrement()`|Pour les classes numériques, opération post-incrémentale atomique équivalente a `value++`|
|`decrementAndGet()`|Pour les classes numériques, opération pré-incrémentale atomique équivalente a `--value`|
|`getAndDecrement()`|Pour les classes numériques, opération post-incrémentale atomique équivalente a `value--`|

#### Exemple d'utilisation
```java
private AtomicInteger sheepCount = new AtomicInteger(0);
private void incrementAndReport() {
    System.out.print(sheepCount.incrementAndGet() + " ");
}
```

```
// Exemples de résultat du programme
2 3 1 4 5 6 7 8 9 10
1 4 3 2 5 6 7 8 9 10
1 4 3 5 6 2 7 8 10 9
```
Les nombres de 1 à 10 seront tous affichés, mais le résultat n'est toujours pas trié.
### Améliorer l'accès avec des blocs *synchronized*
Comment améliorer les résultats pour que chaque *worker* puisse incrémenter et retourner le résultat dans l'ordre ?

La technique la plus commune est d'utiliser un *monitor*, aussi appelé *lock*, pour synchronizer l'accès.

Un *monitor* est une structure qui supporte l'exclusion mutuelle ou la propriété qu'au plus un thread exécute un certain segment de code à un moment précis.

En Java, n'importe quel objet peut être utilisé comme *monitor*, avec le mot-clé *synchronized* comme ceci :
```java
SheepManager manager = new SheepManager();
synchronized(manager) {
    // Travail à faire par un seul thread à la fois
}
```
Cet exemple est appelé « bloc *synchronized* ».

Chaque *thread qui y arrivera vérifiera d'abord s'il n'y a pas un autre *thread* exécutant ce code.

De cette manière, il « obtient le *lock* » du *monitor*.

Si le *lock* est disponible, un seul *thread* entrera dans le bloc, obtenant ainsi le *lock* et empêchant les autres threads de rentrer dans le bloc.

Pendant que le premier thread exécute le bloc, tous les autres threads essaieront d'obtenir le *lock* et attendront que le premier *thread* termine.

Une fois que le thread a fini d'exécuter le bloc, il relâchera le *lock*, permettant a l'un des autres threads de l'exécuter.

```java
// ❌ Ne résout pas le problème, seule la création du thread est synchronisée
for(int i = 0; i < 10; i++) {
    synchronized(manager) {
        service.submit(() -> manager.incrementAndReport());
    }
}
```

```java
// ✔ Résout le problème
import java.util.concurrent.*;

public class SheepManager {
    private int sheepCount = 0;

    private void incrementAndReport() {
        synchronized(this) {
            System.out.println((++sheepCount) + " ");
        }
    }

    public static void main(String[] args) {
        ExecutorService service = null;

        try {
            service = Executors.newFixedThreadPool(20);

            SheepManager manager = new SheepManager()

            for(int i = 0; i < 10 ; i++)
                service.submit(() -> manager.incrementAndReport());            

        } finally {
            if(service != null) service.shutdown();
        }
    }
}
```

Nous aurions pu faire la synchronisation avec n'importe quel objet, du moment que c'est toujours le même objet :
```java
// Exemple avec un objet différent
private final object lock = new Object();

private void incrementAndReport() {
    synchronized(lock) {
        System.out.println((++sheepCount) + " ");
    }
}
```
> Nous aurions aussi pu utiliser une variable atomique avec `synchonized` dans cet exemple mais ce n'est pas nécessaire
> Il n'y a aucune utilité d'utiliser une variable atomique si le seul accès à la variable est fait dans un bloc `synchronized` 

### Synchroniser des méthodes
Dans l'exemple précédent, nous avons utilisé un monitor en utilisant `synchronized(this)` autour du corps de la méthode.

Java fournit une amélioration du compilateur plus pratique pour faire cela.

On peut ajouter le modificateur `synchronized` sur n'importe quelle méthode d'instance pour la synchronizer directement à l'objet.

Par exemple, les deux définitions de méthode suivantes sont équivalentes :

```java
// Avec un bloc synchronized
private void incrementAndReport() {
    synchronized(this) {
        System.out.println((++sheepCount) + " ");
    }
}

// Avec le modificateur synchronized
private synchronized void incrementAndReport() {
    System.out.println((++sheepCount) + " ");
}
```

On peut aussi ajouter le modificateur `synchronized` sur les méthodes statiques. Dans ce cas la classe de l'objet est utilisée comme *monitor*.

Par exemple, les deux définitions de méthode suivantes sont équivalentes pour la synchronisation statique dans notre classe `SheepManager` :
```java
// Avec un bloc synchronized
public static void printDaysWork() {
    synchronized(SheepManager.class) {
        System.out.println("Finished work");
    }
}

// Avec le modificateur synchronized
public static synchronized void printDaysWork { 
    System.out.println("Finished work");
}
```
Vous pouvez utiliser la synchronisation statique si vous avez besoin d'ordonner les accès entre toutes les instances, plutôt qu'avec une seule instance.

### Comprendre le coût de la synchronisation
Bien qu'elle soit pratique, la synchronisation est une pratique coûteuse.

À l'inverse de la programmation *multi-threaded* qui se base sur l'idée de faire plusieurs choses à la fois, la synchronisation se base sur l'idée de prendre plusieurs threads et les faire se comporter d'une manière plus *single-threaded*.

> La synchronisation protège l'intégrité des données au prix des performances.

## Utiliser les collections concurrentes
En plus de gérer les *threads*, l'API Concurrency inclus des classes et des interfaces qui vous permettent de coordonner l'accès aux collections entre les tâches.
### Présentation des collections concurrentes
Pourrions-nous utiliser le mot-clé `synchronized` pour faire la même choses qu; avec nos classes de collection existantes.

oui, nous pourrions faire cela :
```java
public class ZooManager {
    private Map<String, Object> foodData= new HashMap<String, Object>();

    public synchronized void put(String key, String value) {
        foodData.put(key, value);
    }

    public synchronized Object get(String key) {
        return foodData.get(key);
    } 
}
```
Alors, à quoi servent les classes de collections concurrentes ?

Comme `ExecutorService` avec les threads, utiliser les collections concurrentes est extrêmement pratique. Cela nous permet aussi de ne pas introduire des erreurs dans nos implémentations personnalisées.

Enfin, les collections concurrentes contiennent des améliorations de performances qui évitent des synchronisations non nécessaires.

Accéder à des collections depuis plusieurs threads est tellement commun que les auteurs de Java ont décidé qu'il serait bon d'avoir une version alternative de plusieurs des classes de collection normales juste pour avoir de l'accès *multi-threaded*.

Voici un exemple d'implémentation qui n'utilise pas le mot-clé synchronized mais utiliser les classes de collection synchronisées :
```java
public class ZooManager {
    private Map<String, Object> foodData = new ConcurrentHashMap<String, Object>();

    public void put(String key, String value) {
        foodData.put(key, value);
    }

    public Object get(String key) {
        return foodData.get(key);
    }
}
```

Grâce au polymorphisme, même si le type de référence chang, l'objet sous-jacent est toujours une `ConcurrentHashMap`.

Puisque `ConcurrentHashMap` implémente `Map`, elle utilise les mêmes méthodes get() / put(), il n'y a donc pas besoin d'utiliser un type de référence `ConcurrentHashMap` dans cet exemple.

### Comprendre les erreurs de cohérence de la mémoire
L'objectif des classes de collection concurrentes est de résoudre certaines erreurs de cohérence de la mémoire.

Une erreur de cohérence de la mémoire (*memory consistency error*) se produit lorsque deux threads ont deux vues inconsistantes sur ce qui devrait être la même donnée.

Conceptuellement, on souhaite que les écritures d'un thread soient disponibles à un autre thread s'il accède à la collection concurrente après que l'écriture ait été faite.

Lorsque deux threads essaient de modifier la même collection non-concurrentes, la JVM peut lever une `ConcurrentModificationException` à l'exécution. D'autant plus, que cela peut arriver avec un seul thread :

```java
Map<String, Object> foodData = new HashMap<String, Object>();
foodData.put("penguin", 1);
foodData.put("flamingo", 2);
for(String key: foodData.keySet())
    foodData.remove(key);
```
Ce bout de code lèvera une `ConcurrentModificationException` à l'exécution, puisque l'itérateur `keyset()` n'est pas bien mis à jour lorsque le premier élément est supprimé.

Changer la première ligne par une `ConcurrentHashMap` empêchera le code de lancer cette exception à l'exécution :

```java
Map<String, Object> foodData = new ConcurrentHashMap<String, Object>();
foodData.put("penguin", 1);
foodData.put("flamingo", 2);
for(String key: foodData.keySet())
    foodData.remove(key);
```
### Travailler avec les classes concurrentes
Vous savez déjà utiliser la plupart des méthodes disponibles car elles sont un sur-ensemble de celles des classes de collection non concurrentes.

Vous devez utiliser les classes de collection concurrentes à chaque fois que vous avez plusieurs threads qui modifient un objet collection en dehors d'un bloc ou d'une méthode `synchronized`, même si vous ne vous attendez pas à un problème de concurrence.

D'autre part, si ous les threads accèdent à une collection immutable et *read-only* vous n'avez pas besoin d'une collection concurrente.

Il est considéré comme une bonne pratique d'instancier une collection concurrente avec une référence d'interface non-concurrente si possible.

#### Classes de collections concurrentes

|Nom de la classe|Interface du *Java Collection Framework*|Éléments ordonnés ?|triés ?|Bloquants ?|
|---|:---:|:---:|:---:|:---:|
|`ConcurrentHashMap`|`ConcurrentMap`|❌|❌|❌|
|`ConcurrentLinkedDeque`|`Deque`|✔|❌|❌|
|`ConcurrentLinkedQueue`|`Queue`|✔|❌|❌|
|`ConcurrentSkipListMap`|`ConcurrentMap`, `SortedMap`, `NavigableMap`|✔|✔|❌|
|`ConcurrentSkipListSet`|`SortedSet`, `NavigableSet`|✔|✔|❌|
|`CopyOnWriteArrayList`|`List`|✔|❌|❌|
|`CopyOnWriteArraySet`|`Set`|❌|❌|❌|
|`LinkedBlockingDeque`|`BlockingQueue`, `BlockingDeque`|✔|❌|✔|
|`LinkedBlockingQueue`|`BlockingQueue`|✔|❌|✔|

#### Exemples d'utilisation
```java
Map<String, Integer> map = new ConcurrentHashMap<>();
map.put("zebra", 52);
map.put("elephant", 10);
System.out.println(map.get("elephant"));
```

```java
Queue<Integer> queue = new ConcurrentLinkedQueue<>();
queue.offer(31);
System.out.println(queue.peek());
System.out.println(queue.poll());
```

```java
Deque<Integer> deque = new ConcurrentLinkedDeque<>();
deque.offer(10);
deque.push(4);
System.out.println(deque.peek());
System.out.println(deque.pop());
```

La `ConcurrentHashMap` implémente l'interface `ConcurrentMap`, vous pouvez donc utiliser `Map` ou `ConcurrentMap` comme référence pour accéder à une `ConcurrentHashMap` en fonction de si l'appelant à besoin de connaître les détails de l'implémentation. 

### Comprendre les files d'attente bloquantes
#### `LinkedBlockingQueue`
Une `LinkedBlockingQueue` est similaire à une `Queue` sauf qu'elle inclut des méthodes pour attendre un temps spécifié pour compléter une opération.

Comme `BlockingQueue` hérite des méthodes de queue, nous ne présentons que les méthodes spécifiques à l'attente.

|Nom de la méthode|Description|
|---|---|
|`offer(E e, long timeout, TimeUnit unit)`|Ajoute un élément à la queue et attend le temps spécifié, retourne `false` si le temps est écoulé avant que l'espace ne soit disponible|
|`poll(long timeout, TimeUnit unit)`|Récupère et supprime un élément de la queue, attend le temps spécifié et renvoie `null` si le temps est écoulé avant que l'élément ne soit disponible|

Ces deux méthodes peuvent lever une exception *checked* `InterruptedException` puisqu'elles peuvent être interrompues avant que le temps ne soit écoulé, et doivent donc être gérées.

```java
try {
    BlockingQueue<Integer> blockingQueue = new LinkedBlockingQueue<>();

    blockingQueue.offer(39);
    blockingQueue.offer(3, 4, TimeUnit.SECONDS);

    System.out.println(blockingQueue.poll());
    System.out.println(blockingQueue.poll(10, TimeUnit.MILLISECONDS));
} catch (InterruptedException e) {
    // Gérer l'interruption
}
```
#### `LinkedBlockingDeque`
Une `LinkedBlockingDeque` est une classe qui maintient une liste à double entrée et implémente l'interface `BlockingDeque`.

`BlockingDeque` étend `Deque` de la même manière que `BlockingQueue` étend `Queue`.

|Nom de la méthode|Description|
|---|---|
|`offerFirst(E e, long timeout, TimeUnit unit)`|Ajoute un élément au début de la queue et attend le temps spécifié, retourne `false` si le temps est écoulé avant que l'espace ne soit disponible|
|`offerLast(E e, long timeout, TimeUnit unit)`|Ajoute un élément à la fin de la queue et attend le temps spécifié, retourne `false` si le temps est écoulé avant que l'espace ne soit disponible|
|`pollFirst(long timeout, TimeUnit unit)`|Récupère et supprime un élément du debut de la queue, attend le temps spécifié et renvoie `null` si le temps est écoulé avant que l'élément ne soit disponible|
|`pollLast(long timeout, TimeUnit unit)`|Récupère et supprime un élément de la fin de la queue, attend le temps spécifié et renvoie `null` si le temps est écoulé avant que l'élément ne soit disponible|

Ces méthodes peuvent lever une exception *checked* `InterruptedException` puisqu'elles peuvent être interrompues avant que le temps ne soit écoulé, et doivent donc être gérées.

```java
try {
    BlockingDeque<Integer> blockingQueue = new LinkedBlockingDeque<>();

    blockingQueue.offer(91);
    blockingQueue.offerFirst(5, 2, TimeUnit.MINUTES);
    blockingQueue.offerLast(47, 100, TimeUnit.MICROSECONDS);
    blockingQueue.offer(3, 4, TimeUnit.SECONDS);

    System.out.println(blockingQueue.poll());
    System.out.println(blockingQueue.poll(950, TimeUnit.MILLISECONDS));
    System.out.println(blockingQueue.pollFirst(200, TimeUnit.NANOSECONDS));
    System.out.println(blockingQueue.pollLast(1, TimeUnit.SECONDS));
} catch (InterruptedException e) {
    // Gérer l'interruption
}
```

### Comprendre les collections `skipList`
Les classes `SkipList`, `ConcurrentSkipListSet` et `ConcurrentSkipListMap` sont des versions concurrentes de leurs homologues triées, respectivement `TreeSet` et `TreeMap`.

Elles maintiennent leurs éléments ou clé dans leur ordre naturel.

> Lorsque vous voyez `SkipList` ou `SkipSet` à l'examen, pensez « trié »

Comme les autres exemples de queue, il est recommandé d'assigner ces objets à un référence d'interface, comme `SortedMap` ou `NavigableSet`, et de cette manières les utiliser avec le même code qu'au chapitre 3.

### Comprendre les collection `CopyOnWrite`
`CopyOnWriteArrayList` et `CopyOnWriteArraySet` se comportent différemment des autres. Ces classes copient tous leurs éléments dans une structure sous-jacente à chaque fois qu'un élément est ajouté, modifié ou supprimé de la collection.

Par « élément modifié » nous voulons dire que la référence dans la collection est changée. Modifier le contenu de la collection ne causera pas l'allocation d'une nouvelle structure.

Bien que les données soient copiées vers une structure sous-jacente, la référence à l'objet ne change pas.

C'est particulièrement utile dans des environnements *multi-threaded* qui ont besoin d'itérer sur la collection. Chaque itérateur établi avant modification ne verra pas les changements mais itérera sur les éléments de la collection originale.

```java
List<Integer> list = new CopyOnWriteArrayList<>(Arrays.asList(4, 3, 52));

for(Integer item: list) {
    System.out.println(item + " ");
    list.add(9);
}

System.out.println();
System.out.println("Size: " + list.size());
```

```
4 3 52
Size: 6
```
Si nous avions utilisé une `ArrayList` normale, un `ConcurrentModificationException` aurait été levée.

De plus nous évitons de rentrer dans une boucle infinie.

> À proprement parler, les classes `CopyOnWrite` ne suivent pas le pattern « objet immutable »

Les classes `CopyOnWrite` peuvent utiliser beaucoup de mémoire, puisque chaque nouvelle structure de collection doit Être allouée à chaque fois que la collection est modifiée.

Elles sont surtout utilisées dans les environnements multi-threaded où les lectures sont plus courantes que les écritures.

### Obtenir une collection synchronisée
Outre les classes de collections concurrentes que nous avons vues, l'API Concurrency inclut aussi des méthodes pour obtenir des versions synchronisées d'une collection non-concurrente existante.

Ces méthodes, contenues dans la classe `Collections`, contiennent des méthodes *synchronized* qui opèrent sur la collection en entrée et retourne une référence qui a le même type que la collection sous-jacente.

Si vous savez à la création que votre objet à besoin d'être synchronisé, alors vous feriez mieux d'utiliser les classes de collections concurrentes.

Dans le cas ou vous avez une collection qu n'est pas concurrente et a besoin d'être accessible par différents threads, alors utilisez ces méthodes :

|Nom de la méthode|
|---|
|`synchronizedCollection(Collection<T> c)`|
|`synchronizedList(List<T> list)`|
|`synchronizedMap(Map<K, V> m)`|
|`synchronizedNavigableMap(NavigableMap<K, V> m)`|
|`synchronizedNavigableSet(NavigableSet<T> s)`|
|`synchronizedSet(Set<T> s)`|
|`synchronizedSortedMap(SortedMap<T> m)`|
|`synchronizedSortedSet(SortedMap<T> m)`|

Bien que ces méthodes synchronisent l'accès aux éléments de données, comme les méthodes `get()` et `set()`, elles ne synchronisent pas l'accès de tous les itérateurs issus des collections synchronisées.

De ce fait, il est indispensable d'utiliser un bloc de synchronisation si vous avez besoin d'iterer sur n'importe quelle collection retournée

```java
List<Integer> list = Collections.synchronizedList(new ArrayList<>(Arrays.asList(4, 3, 52));

synchronized(list) {
    for(int data: list) System.out.println(data + " ");
}
```

Contrairement aux collections concurrentes, les collections synchronisées lèvent une exception si elles sont modifiées au sein d'un itérateur d'un *thread* unique.
```java
Map<String, Object> foodData = new HashMap<String, Object>();

foodData.put("penguin", 1);
foodData.put("flamingo", 2);

Map<String, Object> synchronizedFoodData = Collections.synchronizedMap(foodData);

for(String key: synchronizedFoodData.keySet())
    synchronizedFoodData.remove(key);
```

Ce code lève une `ConcurrentModificationException` à l'exécution, alors que notre exemple avec `ConcurrentHashMap` non.

À l'exception de pouvoir itérer sur la collection, les objets retournés par ces méthodes fournissent une utilisation par nature *safe* entre plusieurs *threads*.