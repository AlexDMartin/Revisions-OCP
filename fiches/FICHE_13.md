# Concurrence
## Présentation des threads
Un *thread* est la plus petite unité d'exécution qui peut être ordonnancée par le système d'exploitation.

Un processus (*process*) est un groupe de threads associés qui s'exécutent dan le même *environnement partagé*.

Un processus *single-threaded* est un processus qui ne contient qu'un seul thread.

Un processus *multi-threaded* est un processus qui contient plusieurs threads.

Par *environnement partagé*, nous entendons que des threads du même processus utilisent le même espace mémoire et peuvent communiquer directement les uns avec les autres.

Une tâche (*task*) est une unique unité de travail réalisée par un thread.

Un thread peut réaliser plusieurs tâches indépendantes mais toujours une seule à la fois.

### Diagramme de processus
Ce diagramme de processus montre un seul processus avec trois threads, cela montre aussi qu'ils sont assignés à un montant arbitraire `n` de CPUs(*central processing unit*) disponibles dans le système.

### Distinguer les types de threads
Toutes les applications Java sont *multi-threaded*.

#### Threads système
Un « thread système » est est créé par la JVM et s'exécute en arrière-plan de l'application.

Par exemple, le thread de *garbage-collection* est un thread système.

La plupart du temps, l'exécution des threads systèmes est invisible aux yeux du développeur de l'application.

Quand un thread système rencontre un problème et ne peut pas se rétablir (eg. ne plus avoir assez de mémoire) il génère une `Error` et non pas une `Exception`.
#### Threads *user-defined*
Un thread *user-defined* est créé par le développeur de l'application pour accomplir une tâche spécifique.

Dans un souci de simplicité, on dit qu'une application est *single-threaded* lorsqu'elle ne contient qu'un thread *user-defined* (la plupart du temps, les threads systèmes ne nous intéressent pas).
#### *Daemon*
un thread *daemon* est un thread qui n'empêchera pas la JVM de s'arrêter lorsque le programme se termine.

Une application Java se termine lorsque les seuls threads qui tournent encore sont des threads *daemon*. La JVM se fermera automatiquement dans ce cas-là

les threads system et les threads *user-defined* peuvent tous les deux être marqués comme *daemon*.
### Comprendre la concurrence de threads
La propriété d'exécuter des threads et des processus multiples en même temps est appelé « concurrence » (*concurrency*).

Les systèmes d'exploitation utilisent un *thread scheduler* pour déterminer quels threads doivent être en cours d'exécution.

Par exemple un *thread scheduler* put utiliser un *round-robin schedule* ce qui veut dire que chaque thread reçoit une part de cycles CPU égale, avec les threads appelés dans un ordre circulaire.

Lorsque le temps alloué à un thread est terminé mais que le thread n'a pas fini son traitement, il se passe un *context switch*.

Un *context switch* est le procédé qui consiste à sauvegarder l'état du thread pour continuer son execution.

Un thread peut interrompre ou remplacer un autre thread s'il possède une plus haute *thread priority*.

Une *thread priority* est une valeur numérique associée à un thread qui est pris en compte par le *thread scheduler* pour déterminer quels threads doivent être en cours d'exécution.

En Java les *thread priorities* sont des valeurs `integer`, la classe `Thread` définit trois constantes statiques importantes.

Par défaut, les threads *user-defined* reçoivent une priorité de `Thread.NORM_PRIORITY`.

Si vous souhaitez qu'un thread soit exécuté directement, vous pouvez augmenter sa priorité à 6 ou plus.

Si deux threads ont la même priorité, le *thread scheduler* choisira arbitrairement celui à exécuter dans la plupart des situations.

|Constante|Valeur|
|---|:---:|
|`Thread.MIN_PRIORITY`|1|
|`Thread.NORM_PRIORITY`|5|
|`Thread.MAX_PRIORITY`|10|

### Présentation de `Runnable`
`java.lang.Runnable` ou `Runnable` est une interface fonctionnelle qui ne prend pas d'argument et ne retourne pas de données.

La définition de l'interface `Runnable` est :
```java
@FunctionalInterface
public interface Runnable {
    void run();
}
```
L'interface `Runnable` est principalement utilisée pour définir le travail qu'un thread doit exécuter, indépendamment du thread principal.
```java
// Exemple de lambda expression se basant sur l'interface Runnable
() -> System.out.println("Hello world");
() -> {int i = 10; i++;}
() -> {return;}
() -> {}
```
```java
// Exemples de lambda expressions invalides pour l'interface Runnable
() -> ""                        // ❌
() -> 5                         // ❌
() -> {return new Object();}    // ❌
```
#### Créer des classes « Runnable »
Bien que `Runnable` soit devenue une interface fonctionnelle en Java 8, l'interface `Runnable` existait depuis la première version de Java.

Créer une classe qui implémente l'interface `Runnable` était, et est toujours, la façon la plus commune de définir une tache pour un thread.
```java
public class CalculateAverage implements Runnable {
    public void run() {
        // Définir le travail à faire ici
    }
}
```
C'est aussi pratique pour pouvoir passer des informations dans notre objet `Runnable` pour qu'elles soient utilisées dans notre méthode `run()`.
```java
public class CalculateAveragse implements Runnable {
    private double[] scores;

    public CalculateAverages(double[] scores) {
        this.scores = scores;
    }

    public void run() {
        // Définir le travail à faire ici utilisant l'objet "scores"
    }
}
```
Dans ce chapitre, on se concentrera sur la création d'expressions lambda qui implémentent implicitement `Runnable`.

Soyez juste conscient qu'elles sont communément utilisés dans des définitions de classe.
### Création d'un thread
La façon la plus facile d'exécuter un thread est en utilisant la classe `java.lang.Thread` ou `Thread`.

Exécuter une tâche avec un thread est une procédure en deux étapes :
- d'abord vous définissez le `Thread` avec lequel la tâche sera exécutée,
- puis utilisez la méthode `Thread.start()` pour commencer la tâche.

Java ne donne aucune garantie sur l'ordre dans lequel un thread sera traité une fois qu'il a commencé.

> Souvenez-vous que l'ordre n'est pas garanti. L'examen essaiera de vous piéger avec plusieurs tâches démarrées en même temps et vous devrez déterminer le résultat.

Il y a deux façons de définir une tâche / un travail à faire pour une instance de `Thread` :
- Fournir un objet `Runnable` ou une expression lambda au constructeur du `Thread`,
- Créer une classe qui étend `Thread` et *override* la méthode `run()`.

```java
// Exemple de définition de tâche - Runnable
public class PrintData implements Runnable {
    public void run() {
        for(int i = 0; i < 3; i++)
            System.out.println("Printing record: " + i);
    }
    
    public static void main(String[] args) {
        (new Thread(new PrintData())).start();
    }
}
```
```java
// Exemple de définition de tâche - Thread
public class ReadInventoryThread extends Thread {
    public void run() {
        System.out.println("Printing zoo inventory");
    }
    
    public static void main(String[] args) {
        (new ReadInventoryThread()).start();
    }
}
```
À chaque fois que vous créez une instance de Thread, n'oubliez pas de la démarrer avec la méthode `Thread.start()`.
#### Piège - ordre d'exécution
```java
public static void main(String[] args) {
    System.out.println("begin");
    (new ReadInventoryThread()).start();
    (new Thread(new PrintData())).start();
    (new ReadInventoryThread()).start();
    System.out.println("end");
}
```
L'ordre d'exécution ne peut être connu avant l'exécution et les résultats possibles sont multiples. Voici l'un d'entre eux :
```
begin
Printing zoo inventory
Printing record: 0
end
Printing zoo inventory
Printing record: 1
Printing record: 2
```
Bien que l'ordre d'exécution des threads est indéterminé, l'ordre dans un unique thread est linéaire.

À l'examen, faites attention aux cas ou un `Thread` ou un `Runnable` sont créés mais où la méthode `start()` n'est pas appelée.

```java
/* La méthode start n'est pas appelée
*  L'exécution est linéaire, chaque ligne attend que la précédente aie finie de s'exécuter
*/ 
new PrintData.run();
(new Thread(new PrintData())).run();
(new ReadInventoryThread()).run();
```
En général vous ne devriez étendre la classe `Thread` que sous certaines circonstances spécifiques, par exemple lorsque créez votre propre thread *priority-based*.

Dans la plupart des situations vous devriez implémenter l'interface `Runnable` plutôt que d'étendre la classe `Thread`.
#### Pour les entretiens, connaissez les options de création de thread
Voici les raisons pour lesquelles vous devriez favoriser une méthode plutôt qu'une autre :
- Si vous devez définir vos propre règles de `Thread` sur lesquelles plusieurs tâches vont se baser, étendez `Thread`,
- Puisque Java ne supporte pas l'héritage multiple, étendre `Thread` ne vous permet pas d'étendre une autre classe alors qu'implémenter `Runnable` vous permet d'étendre une autre classe,
- Implémenter `Runnable` est souvent une pratique plus « orientée objet » puisqu'elle sépare la tâche du `Thread` la réalisant,
- Implémenter `Runnable` permet à la classe d'être utilisée par plusieurs classes de l'API `Concurrency`.

Vous devriez mentionner le fait qu'on peut maintenant utiliser l'`ExecutorService` pour réaliser des tâches sas avoir à créer des objets `Thread` directement.
## Sonder avec `sleep`
Des fois vous souhaitez qu'un thread sonde le résultat final.

Le sondage (*polling*) est le procédure de vérifier les données par intermittence à intervalles fixes.
```java
// Exemple sans sleep
public class CheckResults {
    private static int counter = 0;
    
    public static void main(String[] args) {
        new Thread(() -> {
            for(int i = 0; i < 500; i++) CheckResults.counter ++;
        }).start();

        while(CheckResults.counter < 100) {
            System.out.println("Not reached yet");
        }
        
        System.out.println("Reached!");
    }
}
```
Utiliser une boucle `while` pur vérifier les données sans notion de délai set considéré comme une très mauvaise pratique puisque cela prend des ressources du processeur pour rien.

On peut améliorer cela en utilisant la méthode `Thread.sleep()` pour implémenter le sondage.
```java
// Exemple sans sleep
public class CheckResults {
    private static int counter = 0;
    
    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {
            for(int i = 0; i < 500; i++) CheckResults.counter ++;
        }).start();

        while(CheckResults.counter < 100) {
            System.out.println("Not reached yet");
            Thread.sleep(1000); // 1 seconde
        }
        
        System.out.println("Reached!");
    }
}
```
C'est beaucoup mieux, le prochain problème que nous allons rencontrer est la variable partagée `counter`. Que se passerait-il si un thread est en train de lire la variable alors qu'un autre est en train de le modifier ? Nous y répondrons plus tard dans ce chapitre.

## Créer des threads avec l'`ExecutorService`
Avec l'annonce de l'API Concurrency, Java à présenté l'`ExecutorService` qui crée et gère les threads pour vous.

D'abord vous obtenez une instance de l'interface `ExecutorService` puis vous envoyez au service les tâches à traiter.

Ce *framework* contient plusieurs fonctionnalités utiles comme le *pooling* et le *scheduling*.

Il est le choix recommandé si vous souhaitez exécuter un tâche séparément, même si vous n'avez besoin que d'un seul thread.

### Présentation du *Single Thread Executor*
L'API Concurrency contient la classe *factory* `Executor` qui peut être utilisée pour créer des instances d'objets `ExecutorService`
#### Exemple d'utilisation de `newSingleThreadExecutor()` pour obtenir un `ExecutorService`
```java
import java.util.concurrent.*;

public class ZooInfo {
    public static void main(String[] args) {
        ExecutorService service = null;

        try {
            service = Executors.newSingleThreadExecutor();

            System.out.println("begin");

            service.execute(() -> System.out.println("Printing zoo inventory"));
            service.execute(() -> {
                for(int i = 0; i < 3; i++) 
                    System.out.println("Printing record:" + i);
            });
            service.execute(() -> System.out.println("Printing zoo inventory"));
        
            System.out.println("end");
        } finally {
            if(service != null) service.shutdown();
        }
    }
}
```
Avec un single thread executor, les résultats sont garantis d'être dans l'ordre de leur ajout dans l '`ExecutorService`.
```
begin
Printing zoo inventory
Printing record: 0
Printing record: 1
end
Printing record: 2
Printing zoo inventory
```
`end` est affiché alors que nos threads sont encore en train de tourner car la méthode `main()` est un thread indépendant de l'`ExecutorService`
#### Exécution de plusieurs tâches
Dans l'exemple précédent, il est possible que les trois tâches soient soumise à l'exécution avant que la première tâche n'aie commencée. Dans ce cas-là le *single thread executor* les mettra en file d'attente et attendra que la tâche précédente se termine avant de lancer la suivante.
 
Bien que les tâches soient garanties d'être exécutées dans l'ordre dans lequel ils ont été soumis pour un *single thread executor*, il faut éviter de s'appuyer sur ce comportement pour ordonner des évènements. Comme nous le verrons plus tard, lorsque l'on augmente le nombre de threads, cette garantie est perdue.
### Arrêter un *thread executor*
Une fois que vous avez fini d'exécuter un thread executor, il est important d'appeler la méthode `shutdown()`.

Un *thread executor* crée un thread non-daemon à la première tâche exécutée, en cas de manquement d'appel à la méthode `shutdown()` votre application ne se terminera jamais.

Le processus de shutdown commence par rejeter toutes les nouvelles tâches soumises au thread executor en continuant à exécuter les tâches soumises précédemment. Pendant ce temps, la méthode `isShutdown()` retournera `true` mais la méthode isTerminated retournera `false`.

Si une nouvelle tâche est soumise pendant que le *thread executor* est en train de se fermer, une `RejectedExecutionException` sera levée.

Une fois que toutes les tâches ont été complétées, `isShutdown()` et `isTerminated()` retourneront `true` 

Pour l'examen, vous devez savoir que la méthode `shutdown()` ne stoppe aucune tâche.

Nous disposons de la méthode `shutdownNow()` qui stoppe toutes les tâches en cours d'exécution et ignore toutes celles qui n'ont pas encore commencé.

Il est possible de créer un thread qui ne terminera jamais, et toutes les tentatives d'interruption seront ignorées.

`shutdownNow()` renvoie une `List<Runnable>` des tâches qui ont été soumises au *thread executor* mais qui n'ont jamais démarré.

#### Arrêter un *thread executor* avec `finally`
L'interface `ExecutorService` n'implémente pas `AutoCloseable` donc vous ne pouvez pas utiliser l'instruction *try-with-resources*.

Vous pouvez utiliser un block `finally`. Bien que ce ne soit pas nécessaire, c'est considéré comme une bonne pratique.

```java
ExecutorService service = null;
try {
    service = Executors.newSingleThreadExecutor();
    // Ajout de tâches au thread executor
} finally {
    if(service != null) service.shutdown();
}
```
Cette solution est pratique pour les *thread executor* dont vous vous débarrassez après les avoir utilisés. Il n'est pas valable pour ceux qui sont persistant pendant la vie de l'application.

Par exemple, vous pourriez avoir une instance de *thread executor* `static` et le partager entre tous les *process*.

Dans ce cas-là vous définiriez une méthode statique pouvant être appelée à n'importe quel moment permettant à l'utilisateur de signaler qu'il souhaite fermer le programme.

Souvenez-vous quá défaut d'éteindre un thread executor après qu'au moins un thread ait été créé résultera en un programme ne se terminant jamais.

### Soumettre des tâches
#### `execute()`
La première méthode présentée est `execute()`, elle est héritée de l'interface `Executor` que l'interface `ExecutorService` étend.

La méthode `execute()` prend une expression lambda ou une instance de `Runnable` et complète la tâche de façon asynchrone. 

Puisque la méthode `execute()` renvoie `void()`, cela ne nous dit rien sur le résultat de la tâche.

#### `submit()`
`ExecutorService` possède la méthode `submit()` qui, comme `execute()`, permet de completer des tâches de façon asynchrone.

Contrairement à `execute()` submit retourne un objet `Future` qui peut être utilisé pour déterminer si la tâche est complétée ou renvoyer un objet générique une fois qu'elle est complétée.

#### Méthodes d'`ExecutorService`
<table>
<thead>
  <tr>
    <th>Nom de la méthode</th>
    <th>Description</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td><code>void execute(Runnable command)</code></td>
    <td>Exécute une tâche <code>Runnable</code> à un moment donné dans le futur</td>
  </tr>
  <tr>
    <td><code>Future&lt;?&gt; submit(Runnable task)</code></td>
    <td>Exécute une tâche <code>Runnable</code> à un moment donné dans le futur et retourne une <code>Future</code> représentant la tâche</td>
  </tr>
  <tr>
    <td><code>&lt;T&gt; Future&lt;T&gt; submit(Callable&lt;T&gt; task)</code></td>
    <td>Exécute une tâche <code>Runnable</code> à un moment donné dans le futur et retourne une <code>Future</code> représentant les résultats en cours de la tâche</td>
  </tr>
  <tr>
    <td><code>&lt;T&gt; List&lt;Future&lt;T&gt;&gt; invokeAll(Collection&lt;? extends Callable&lt;T&gt;&gt; tasks) throws InterruptedException</code></td>
    <td>Exécute les tâches données, retourne les résultats de toutes les tâches de façon synchrone comme une <code>Collection</code> d'objet <code>Future</code> dans l'ordre dans lequel elles étaient dans la collection originelle</td>
  </tr>
  <tr>
    <td><code>&lt;T&gt; T invokeAny(Collection&lt;? extends Callable&lt;T&gt;&gt; tasks) throws InterruptedException, ExecutionException</code></td>
    <td>Exécute les tâches données, retourne le résultat de l'une des tâche terminée et annule les tâches non terminées</td>
  </tr>
</tbody>
</table>
En pratique la méthode `submit()` est similaire à la méthode `execute()` à part que `submit()` renvoie un objet `Future` qui peut être utilisé pour déterminer si la tâche a complété son exécution.

#### Soumettre des tâches : `execute()` vs `submit()`
Les méthodes `submit()` et `execute()` sont quasiment identiques lorsqu'elles sont appliquées à des expressions `Runnable` .

La méthode `submit()` à l'avantage de faire la même chose qu'`execute()`, mais retourne un objet qui peut être utilisé pour suivre le résultat.

À cause de cela et du fait qu'`execute()` ne supporte pas les expressions `Callable`, nous préferons utiliser `execute()` même si nous ne stockons pas la référence à l'objet `Future`.

Pour l'examen, vous devez être habitué à `execute()` et `submit()` mais dans votre code nous vous recommandons d'utiliser `submit()` plutôt qu'`execute()`. 

### Soumettre une collection de tâches
Les deux méthodes à connaître pour la soumission de collection de tâches à l'examen sont `invokeAll()` et `invokeAny()`.

Ces deux méthodes prennent un objet `Collection` contenant une liste de tâches à exécuter.

Ces deux méthodes s'exécutent de façon synchrone, c'est-à-dire que, différemment des méthodes utilisées pour soumettre des tâches à un *thread executor*, elles attendront que les résultats soient disponibles avant de rendre le contrôle au programme l'appelant.

#### `invokeAll()`
La méthode `invokeAll()` exécute toutes les tâches données dans la collection et retourne une liste ordonnée d'objets `Future`, avec un objet `Future` correspondant à chaque tâche terminée, dans l'ordre dans lequel elles étaient dans la collection originelle.

La méthode `invokeAll()` attendra à l'infini que toutes les tâches soient terminées.
#### `invokeAny()`
La méthode `invokeAny()` exécute une collection de tâche et retourne le résultat de l'une des tâches qui a complété son exécution avec succès, puis annule toutes les tâches non terminées.

Bien que la première tâche est souvent celle qui est retournée, ce comportement n'est pas garanti, car n'importe quelle tâche terminée avec succès peut être retournée.

La méthode `invokeAny()` attendra à l'infini qu'au moins l'une des tâches soit terminée.

> L'interface `ExecutorService` inclut des versions surchargées des méthodes `invokeAll()` et `invokeAny()` prenant en paramètre une valeur de timeout et un paramètre `TimeUnit`.

### Attendre les résultats
la méthode `submit()` renvoit un objet `java.util.concurrent.Future<V>` qui peut être utilisé pour determiner le résultat
```java
// Exemple d'utilisation de submit
Future<?> future = service.submit(() -> System.out.println("Hello zoo"));
```

#### Méthodes de la classe `Future`
|Nom de la méthode|Description|
|---|---|
|`boolean isDone()`|Retourne true si la tâches est complétée, a levé une exception, ou a été annulée|
|`boolean isCancelled()`|Retourne true si la tâche a été annulée avant qu'elle ce soit complétée normalement|
|`boolean cancel()`|Tente d'annuler l'exécution de la tâche|
|`V get()`|Récupère le résultat de la tâche, attend à l'infini s'il n'est pas disponible directement|
|`V get(long timeOut, TimeUnit unit)`|Récupère le résultat de la tâche en attendant le temps spécifi. Si le résultat n'est pas disponible avant le timeout, une exception *checked* `TimeoutException` est levée|

#### Exemple d'utilisation de Future
```java
import java.util.concurrent.*;

public class CheckResults {
    private static int counter = 0;
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        ExecutorService service = null;

        try {
            service = Executors.newSingleThreadExecutor();

            Future<?> result = service.submit(() -> {
                for(int i; i < 500; i++) CheckResults.counter++;
            });

            result.get(10, TimeUnit.SECONDS);
            
            System.out.println("Reached");
        } catch (TimeoutException e) {
            System.out.println("Not reached in time");
        } finally {
            if(service != null) service.shutdown();
        }
    }
}
```

#### `TimeUnit`
La méthode `get()` prend en paramètre une valeur optionnelle et une enum de type `java.util.concurrent.TimeUnit`.

|Nom de l'enum|Description|
|---|---|
|`TimeUnit.NANOSECONDS`|Temps en un milliardième de seconde (1/1.000.000.000)|
|`TimeUnit.MICROSECONDS`|Temps en un millionième de seconde (1/1.000.000)|
|`TimeUnit.MILLISECONDS`|Temps en un millième de seconde (1/1.000)|
|`TimeUnit.SECONDS`|Temps en secondes|
|`TimeUnit.MINUTES`|Temps en minutes|
|`TimeUnit.HOURS`|Temps en heures|
|`TimeUnit.DAYS`|Temps en jours|

### Présentation de `Callable`
Lorsque l'API Concurrency a été publiée en Java 5, la nouvelle interface `java.util.concurrent.Callable` a été ajoutée.

Elle est similaire à `Runnable` sauf que sa méthode `call()` retourne une valeur et lève une exception *checked* alors que la méthode `run()` de `Runnable` renvoie `void` et ne peut pas lever d'exception *checked*.

Avec `Runnable`, `Callable` est devenu une interface fonctionnelle en java 8.

Voici la définition de l'interface `Callable` :
```java
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```

L'interface `Callable` a été présentée comme une alternative à l'interface `Callable`, puisqu'elle permet de récupérer plus facilement plus de détails sur la tâche après sa completion.

L'`ExecutorService` inclus une version surchargée de `submit()` qui prend un objet `Callable` et retourne un objet générique `Future<T>`.

#### Expression lambda Ambigüe : `Callable` vs `Supplier`
Les deux ne prennent pas d'argument et renvoient un type générique. La seule différence est que `Callable` peut lever une exception *checked*.

Des fois, vous ne pourrez pas discerner les deux.
```java
public class AmbiguousLambdaSample {
    public static void useCallable(Callable<Integer> expression) {}
    public static useSupplier(Supplier<Integer> expression) {}

    public static void use(Supplier<Integer> expression) {}
    public static void use(Callable<Integer> expression) {}
    
    public static void main(String[] args) {
        useCallable(() -> { throw new IOException(); });    // ✔ COMPILE
        useSupplier(() -> { throw new IOException(); });    // ❌ NE COMPILE PAS, Supplier ne peut pas lever une exception checked
        use(() -> { throw new IOException(); });            // ❌ NE COMPILE PAS, le compilateur ne sait pas quoi faire
    }
}
```
Cette ambiguïté peut être résolue avec un cast explicite.
```java
use((Callable<Integer>) () -> { throw new IOException(); });    // ✔ COMPILE
```

#### `get()`
Contrairement à `Runnable`, où la méthode `get()` retourne toujours `null`, la méthode get() d'un objet `Future` renvoie le type générique associé ou `null`.
```java
// Exemple d'utilisation de get()
import java.util.concurrent.*;

public class AddData {
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        ExecutorService service = null;

        try {
            service = Executors.newSingleThreadExecutor();

            Future<Integer> result = service.submit(() -> 30 + 11);

            System.out.println(result.get());
        } finally {
            if(service != null) service.shutdown();
        }
    }
}
```
Puisque `Callable` supporte un type de retour lorsqu'il est utilisé avec `ExecutorService`, elle est préférée par rapport à `Runnable` lors de l'utilisation de l'API Concurrency.

#### Exceptions *checked* avec `Callable` et `Runnable`
L'interface Callable supporte les exceptions *checked* alors que l'interface `Runnable` ne la supporte pas sans *try / catch*.

```java
// Thread.sleep() lève une exception checked InterruptedException

service.submit(() -> {Thread.sleep(1000); return null;});   // ✔ COMPILE, traité comme un Callable
service.submit(() -> {Thread.sleep(1000);});                // ❌ NE COMPILE PAS, traité comme un Runnable
```

### Attendre que toutes les tâches soient terminées
Après avoir soumis un ensemble de tâches a un *thread executor*, il est commun d'attendre les résultats.

Une solution est d'appeler la méthode `get()` sur chaque objet `Future` retourné parla méthode `submit()`.

Si nous n'avons pas besoin des résultats, il existe une approche bien plus simple :
- d'abord on éteint le thread executor en appelant la méthode `shutdown()`,
- puis on utilise la méthode `awaitTermination(long timeout, TimeUnit unit)`, accessible à tous les *thread executors*.

La méthode `awaitTermination` attend le temps spécifié et retourne plus tôt si toutes les tâches sont terminées ou si une `InterruptedException` est détectée.

```java
ExecutorService service = null;
try {
    service = Executors.newSingleThreadExecutor();

    // Ajouter des tâches au thread executor 
} finally {
    if(service != null) service.shutdown()
}

if(service != null) {
    service.awaitTermination(1, TimeUnit.MINUTES);
    // Check si toutes les tâches sont terminées
    if(service.isTerminated())
        System.out.println("All tasks finished");
    else
        System.out.println("At least one task is still running");
}
```

### Planification des tâches
L'interface `ScheduledExecutorService`, qui est une sous-interface d'`ExecutorService` qui peut être utilisée pour planifier des tâches qui vont se produire dans un temps futur.

Comme `ExecutorService`, on obtient une instance de `ScheduleExecutorService` en utilisant une méthode *factory* dans la classe `Executors` :
```java
ScheduledExectorService service = Executors.newSingleThreadScheduledExecutor();
```
Nous pourrions aussi caster implicitement une instance de `ScheduledExecutorService` en `ExecutorService` mais nous perdrions les méthodes de planification que nous souhaitons utiliser.

|Nom de la méthode|Description|
|---|---|
|`schedule(Callable<V> callable, long delay, TimeUnit unit)`|Crée et exécute une tâche `Callable` après le délai donné|
|`schedule(Runnable command, long delay, TimeUnit unit)`|Crée et exécute une tâche `Runnable` après le délai donné|
|`scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit)`|Crée et exécute une tâche `Runnable` après le délai initial donné, puis crée une nouvelle tâche après chaque période qui passe|
|`scheduleAtFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit)`|Crée et exécute une tâche `Runnable` après le délai initial donné puis ultérieurement avec le délai donné entre la fin de l'exécution et le commencement de la suivante|

Ces méthodes sont très pratiques dans l'API Concurrency, car elles permettent de réaliser des tâches très complexes en une seule ligne de code.

Les paramètres de délais et les périodes se basent sur l'argument TimeUnit pour déterminer le format de la valeur
#### `schedule()`
Qu'elle soit utilisé avec `Callable` ou `Runnable`, elle retourne une `ScheduleFuture<V>`, similaire à `Future<V>` sauf qu'elle inclut la méthode `getDelay()` pour savoir quel a été le délai donné à la création du processus.

```java
ScheduledExecutorService service = Executors.newSingleThreadScheduledExecutor();

Runnable task1 = () -> System.out.println("Hello zoo");
Callable<String> task2 = () -> "Monkey";

Future<?> result1 = service.schedule(task1, 10, TimeUnit.SECONDS);  // Prévue dans 10 secondes
Future<?> result2 = service.schedule(task2, 8, TimeUnit.MINUTES);   // Prévue dans 8 minutes
```
#### `scheduleAtFixedRate()`
Crée une nouvelle tâche et la soumet à l'*executor* à chaque période, sans prendre en compte le fait que la précédente soit terminée ou non.

```java
service.scheduleAtFixedRate(command, 5, 1, TimeUnit.MINUTES); // Toutes les minutes après les 5 minutes de delai initial
```
### `scheduleAtFixedDelay()`
Crée une nouvelle tâche après que la précédente soit terminée.
```java
service.scheduleAtFixedDelay(command, 0, 2, TimeUnit.MINUTES); // Toutes les deux minutes une fois que la tâche est terminée
```
> Souvenez vous que ni `scheduleAtFixedDelay()` ni `scheduleAtFixedRate()` ne prennent une objet `Callable` comme paramètre

Puisque ces tâches sont planifiées pour être exécutées à l ínfini, tant que le `ScheduledExecutor` tourne, elles généreront une série d'objets `Future` à l'infini.

### Augmenter la concurrence avec les *pools*
Un *thread pool* est un groupe de thread pré-instanciés et réutilisables qui sont disponibles pour effectuer un ensemble de tâches arbitraires.

#### Méthodes d'`Executors`
|Nom de la méthode|Type de retour|Description|
|---|---|---|
|`newSingleThreadExecutor()`|`ExecutorService`|Crée un *single-threaded executor* qui utilise un unique thread *worker* qui opère sur une file d'attente sans limite. Les résultats sont traités séquentiellement dans l'ordre dans lequel ils sont soumis|
|`newSingleThreadScheduledExecutor()`|`ScheduledExecutorService`|Crée un *single-threaded executor* qui peut planifier les commandes por être exécutées selon un délai ou une période précise|
|`newCachedThreadPool()`|`ExecutorService`|Crée un *thread pool* qui crée de nouveaux threads si besoin, mais réutilisera les threads déjà construits si besoin|
|`newFixedThreadPool(int nThreads)`|`ExecutorService`|Crée un *thread pool* qui réutilisera un nombre donné de threads et qui opéreront sur un file d'attente sans limite partagée|
|`newScheduledThreadPool(int nThreads)`|`ScheduledExecutorService`|Crée un *thread pool* qui peut planifier des commandes pour être exécutées avec un certain délai ou périodiquement|

La différence entre un *single-thread* et un *pool-thread executor* est ce qu'il se passe lorsqu'une tâche est déjà en train de tourner :
- un *single-thread executor* attendra qu'un thread soit disponible avant de lancer les prochaines tâches
- un *pool-thread executor* peut exécuter la prochaine tâche de façon concurrente. Si le pool n'a plus de *thread* disponible, alors la tâche sera ajoutée à la file d'attente par le *thread executor* et sera donc en attente d'être complétée.

#### `newCachedThreadPool()`
La méthode `newCachedThreadPool()` créera un thread pool de taille illimitée et allouera un nouveau thread à chaque fois que l'on en a besoin et que les autres *threads* sont occupés.

Elle est souvent utilisée pour les *pools* qui nécessitent d'exécuter beaucoup de tâches asynchrones de courte durée.

Pour les processus de longue durée, l'utilisation de cette méthode est fortement déconseillée.
#### `newFixedThreadPool()`
La méthode `newFixedThreadPool()` prend un nombre de *threads* et les alloues à leur création.

Du moment que le nombre de tâches est inférieur au nombre de tâches, toutes les tâches seront exécutées de façon concurrente.

Si à n'importe quel moment, le nombre de tâches dépasse le nombre de *threads* dans le pool, elles attendront de la même manière qu'avec un *single-thread executor*.

En réalité, appeler `newFixedThreadPool(1)` revient à appeler `newSingleThreadExecutor()`.
#### `newScheduledThreadPool()`
La méthode `newScheduledThreadPool()` est similaire à `newFixedThreadPool()` sauf qu'elle retourne une instance de `ScheduledExecutorService` et que par conséquent elle est compatible avec la planification de tâches.

Cet *executor* a une subtile différence dans la manière de laquelle `scheduleAtFixedRate()` opère :
```java
ScheduledExecutorService service = Executors.newScheduledThreadPool(10);

service.scheduleAtFixedRate(command, 3, 1, TimeUnit.MINUTES);
```
Si le pool est suffisamment large pour que chaque *thread* se finisse et retourne dans le pool, il y aura toujours assez de thread.

#### Choisir la taille du *pool*
En pratique, il peut être délicat de choisir la taille d'un pool.

En général, vous souhaitez au minimum avoir quelques *threads* en plus que ce que vous pensez avoir réellement besoin.

D'un autre côté, vous ne voulez pas choisir tellement de threads que votre application prendra trop de ressources ou trop de puissance de traitement de votre CPU.

Le nombre de CPUs disponibles est souvent utilisé pour déterminer la taille des *threads*. On peut l'obtenir comme ceci :
```java
Runtime.getRuntime().availableProcessors();
```

