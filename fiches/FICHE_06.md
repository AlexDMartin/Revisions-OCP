## `Comparator` vs `Comparable`
Le sujet de l'ordre a été évoqué dans les parties `TreeSet` et `TreeMap`.

Pour les nombres, on utilise l'ordre numérique.

Pour les `String` l'ordre est selon la cartographie des caractères unicode. d'abord les nombres, puis les lettres en majuscule, puis les lettres en minuscule.

Java vous fournit l'interface `Comparable` qui vous permet de trier les objets que vous créez.

Il existe aussi une classe `Comparator` qui est utilisée pour spécifier que vous voulez trier dans un ordre différent que celui qui est déjà intégré.

L'examen essaiera de vous piéger à confondre entre `Comparable` et `Comparator`.
### `Comparable`
L'interface `Comparable` ne possède qu'une seule méthode. Voici son implémentation complète :
```java
public interface Comparable<T> {
    public int compareTo(T o);
}
``` 
L'utilisation des génériques vous permet d'éviter le cast lorsque vous implémentez `Comparable`.

Types de retours de la méthode `compareTo()` :
- le nombre zéro est retourné lorsque l'objet actuel est égal à l'objet en argument de `compareTo()`,
- un nombre plus petit que zéro est retourné lorsque l'objet actuel est plus petit que l'objet en argument de `compareTo()`,
- un nombre plus grand que zéro est retourné lorsque l'objet actuel est plus grand que l'objet en argument de `compareTo()`.

> Rappelez vous que `id - a.id` trie les éléments dans un ordre ascendant et `a.id - id` trie les éléments dans un ordre descendant.

Lorsque vous travaillez avec du code legacy, `compareTo` requiert un cast puisque Object est passé (pas de génériques).

Vous êtes encouragés à faire que vos méthodes `compareTo`.

> `Comparable` est-elle une interface fonctionnelle ?
> Techniquement oui, mais il serait bizarre d'utiliser des lambdas pour `Comparable`
> L'objectif de comparable est d'être implémenté à l'interieur de l'objet comparé

### Comparator
Des fois, vous voulez trier un objet qui n'implémente pas `Comparable` ou voulez trier des objets différemment à différentes occasions.

`Comparator` est une interface fonctionnelle, ce qui veut dire qu'elle ne possède qu'une méthode abstraite à implémenter.

```java
// Exemple
Comparator<Duck> byWeight = (d1, d2) -> d1.getWeight() - d2.getWeight();
```
L'examen essaiera de vous piéger, apprenez vraiment ce tableau par coeur :

|Différence|`Comparable`|`Comparator`|
|---|---|---|
|Package name|`java.lang`|`java.util`|
|L'interface doit être implémentée par la classe comparante|✔|❌|
|Nom de la méthode dans l'interface|`compareTo`|`compare`|
|Nombre de paramètres|1|2|
|Il est commun de les déclarer avec une lambda|❌|✔|

> Attention aux `null` lorsque vous faites des comparaisons.
> Il est commun de trier les `null` avant toutes les autres valeurs.
## Rechercher et trier
Si une classe n'implémente pas `Comparable`, appeler `sort` ne compilera pas.

Vous pouvez éviter ce prolème en passant un `Comparator` à `sort()`.

`sort()` et `binarySearch()` vous permettent de passer un `Comparator` si vous ne voulez pas utiliser l'ordre de tri « naturel ».

> Attention à ne pas briser la précondition de `binarySearch` avec `Comparator`, les objets doivent être triés dans l'ordre ascendant (du plus petit au plus grand).

> Attention lorsque vous essayez d'insérer un élément dans un `Set` il faut qu'il soit comparable, sinon vous aurez une `ClassCastException`.

Vous pouvez spécifier que vous souhaitez trier dans un ordre particulier à la création de la collection. Par exemple :

```java
Set<Rabbit> rabbits = new TreeSet<>(new Comparator<Rabbit>(){
    public int compare(Rabbit r1, Rabbit r2) {
        return r1.id = r2.id;
    }
});
```
## Ajouts de Java 8
### Les références de méthodes
Les références de méthode sont un moyen de rendre votre code plus court en réduisantla quantité de code qui peut être déduit en mentionnant simplement le nom de la méthode.

Exemple avec une lambda :
```java
Comparator<Duck> byWeight = (d1, d2) -> DuckHelper.compareByWeight(d1, d2);
```
Il y a un peu de redondance. La lambda ne fait rien d'autre que prendre deux paramètres et les transmettre à une autre méthode.

L'opérateur `::` dit a Java de les passer automatiquement a cette méthode.
```java
Comparator<Duck> byWeight = DuckHelper::compareByWeight;
```
> `DuckHelper::compareByWeight` renvoie une interface fonctionnelle et non pas un `int`.

Il y a 4 types de références de méthode :
- les méthodes statiques,
- les méthodes d'instance sur une instance particulière,
- les méthodes d'instance sur une instance définie à l'exécution,
- Constructeurs.

`Predicate` est une interface fonctionnelle qui prend un seul paramètre de n'importe quel type et retourne un booléen.

`Consumer` est une interface fonctionnelle qui prend un seul paramètre de n'importe quel type et à un type de retour `void`.

`Supplier` est une interface fonctionnelle qui ne prend pas de paramètre et retourne n'importe quel type.
#### Méthodes statiques
```java
Consumer<List<Integer>> methodRef1 = Collections::sort;
Consumer<List<Integer>> lambda1 = l -> Collections.sort(l);
```
Comment Java sait quelle surcharge de `sort()` appeler ? Il se base sur le contexte

Dans cet exemple on demande un `Consumer` donc java sait laquelle appeler
#### Méthodes d'instance sur une instance particulière
```java
String str = "abc";
Predicate<String> methodRef2= str::startsWith;
Predicate<String> lambda2 = s -> str.startWith(s);
```
#### Méthodes d'instance sur une instance définie à l'exécution
```java
Predicate<String> methodRef3= String::isEmpty;
Predicate<String> lambda3 = s -> s.isEmpty();
```
#### Constructeurs
```java
Supplier<ArrayList> methodRef4= ArrayList::new;
Supplier<ArrayList> lambda= () -> new ArrayList();
```

### Supprimer conditionnellement (`removeIf`)
Java 8 a introduit une nouvelle méthode appelé `removeIf`. Avant cela, nous pouvions supprimer un élément d'une collection ou index specifique d'une liste.

Maintenant nous pouvons spécifier ce que l'on souhaite supprimer grace à un block de code. 
```java
boolean removeIf(Predicate<? super E> filter)
```
`removeIf` utilise un `Predicate`, c'est-à-dire une lambda qui prend un seul paramètre et renvoie un booléen.

Les lambdas utilisent l'exécution différée, cela nous permet de spécifier la logique à exécuter quand ce point du code est atteint.

Vous devez juste retenir que `removeIf` est l'une des deux méthodes des collections qui prend en paramètre une *lambda*.

### Mettre à jour tous les éléments
Une autre introduite dans les listes est `replaceAll`. Java 8 vous permet d'y passer une expression lambda qui sera appliquée à tous les éléments de la liste.

Le résultat remplace la valeur actuelle de l'élément.
```java
void replaceAll(UnaryOperator<E> o)
```
`replaceAll` utilise un `UnaryOperator`, qui prend un seul paramètre et renvoie une valeur du même type.
### Boucler sur une collection (`foreach`)
Nous pouvons utiliser un `iterator`, une boucle `for` avancée ou d'autres approches. Nous pouvons aussi utiliser les lambdas de Java 8.
```java
List<String> cats = Arrays.asList("Annie", "Ripley");
for(String cat: cats) System.out.println(cat);
```
peut être converti en :
```java
// Utilisation d'un Consumer qui prend un seul paramètre et ne retourne rien.
cats.forEach(c -> System.out.println(c));
```
voir même :
```java
cats.forEach(System.out::println);
```
### Utiliser la nouvell API `Map` de Java 8
Java 8 à ajouté beaucoup de nouvelles méthodes à l'interface `Map`. Seule `merge()` est à connaître pour l'OCP.

`computeIfPresent()` et `computeIfAbstent()` sont ajoutées pour l'examen de mise à niveau.

Des fois, vous devez mettre à jour la valeur d'une clé spécifique dans la map. Il y a plusieurs moyens d'y parvenir :
#### Sans condition
```java
Map<String, String> favorites = new HashMap<>();
favorites.put("Jenny", "Bus Tour"); 


favorites.put("Jenny", "Tram");
System.out.println(favorites); // { Jenny: Tram }
```
#### Conditionnellement
Il existe une autre méthode, appelée `putIfAbsent()`, qui vous permet de determiner une valeur dans la map, mais cette méthode saute les valeurs non nulles.
```java
Map<String, String> favorites = new HashMap<>();
favorites.put("Jenny", "Bus Tour"); 
favorites.put("Tom", null); 


favorites.putIfAbsent("Jenny", "Tram");
favorites.putIfAbsent("Sam", "Tram");
favorites.putIfAbsent("Tom", "Tram");
System.out.println(favorites); // { Tom: Tram, Jenny: Bus Tour, Sam: Tram }
```

Des fois vous aurez besoin de plus de logique pour déterminer quelle valeur utiliser :
### `merge`
La méthode `merge()` permet d'ajouter de la logique pour indiquer quelle valeur choisir .
#### Utilisation normale
```java
BiFunction<String, String, String> mapper = (v1, v2) -> v1.length() > v2.length() ? v1 : v2;

Map<String, String> favorites = new HashMap<>();
favorites.put("Jenny", "Bus Tour"); 
favorites.put("Tom", "Tram");

String jenny = favorites.merge("Jenny", "Skyride", mapper);
String tom = favorites.merge("Tom", "Skyride", mapper);

System.out.println(favorites); //  { Tom: Skyride, Jenny: Bus Tour }
System.out.println(jenny); // Bus Tour
System.out.println(tom); // Skyride
```

Une `BiFonction` est une interface fonctionnelle. Dans ce cas, elle prend deux paramètres du même type et renvoie une valeur de ce type.

Cette implémentation retourne celle avec le nom le plus long : 
- "Bus Tour" est plus long que "Skyride", donc ça garde "Bus Tour" pour Jenny
- "Skyride" est plus long que "Tram", donc la valeur de Tom est mise à jour avec "Skyride"
#### Gestion des `null` et des clés manquantes
La méthode `merge()` ajoute aussi une logique pour la gestion des `null` et la gestion des clés manquantes  
```java
BiFunction<String, String, String> mapper = (v1, v2) -> v1.length() > v2.length() ? v1 : v2;

Map<String, String> favorites = new HashMap<>();
favorites.put("Sam", null);

favorites.merge("Tom", "Skyride", mapper);
favorites.merge("Sam", "Skyride", mapper);

System.out.println(favorites); // { Tom: Skyride, Sam: Skyride }
```
Le mapper n'est tout simplement pas appelé, s'il l'était une `NullPointerException` serait lancée.

Le mapper n'est utilisé que quand il y a un choix à faire entre deux valeurs.
#### Suppression grâce à `null`
Quand le mapper renvoie null, la clé est tout simplement supprimée
```java
BiFunction<String, String, String> mapper = null;

Map<String, String> favorites = new HashMap<>();
favorites.put("Jenny", "Bus Tour");
favorites.put("Tom", "Bus Tour");

favorites.merge("Jenny", "Skyride", mapper);
favorites.merge("Sam", "Skyride", mapper);

System.out.println(favorites); // { Tom: Bus Tour, Sam: Skyride }
```
- Tom est laissé tout seul car aucun appel à `merge` n'a été fait sur lui.
- Jenny a été supprimée car la fonction merge a renvoyé `null`.
- Sam a été ajouté car la clé n'était pas dans la map à l'origine (pas d'appel au mapper, car pas de comparaison).

### `computeIfPresent` et `computeIfAbsent`
Ces deux méthodes sont à l'examen de mise à niveau.

En gros, `computeIfPresent()` appelle la `BiFonction` si la clé est trouvée et `computeIfAbsent()` l'appelle si elle ne l'est pas ou si elle est `null`.

### Récapitulatif, `merge`, `computeIfPresent`, `computeIfAbsent` 

|Scenario|`merge`|`computeIfAbsent`|`computeIfPresent`|
|---|---|---|---|
|La clé est déjà dans la map|Résultat de la fonction|Pas d'action|Résultat de la fonction|
|La clé n'est pas dans la map|Ajoute une nouvelle valeur dans la map|Résultat de la fonction|Pas d'action|
|Interface Fonctionnelle Utilisée|`BiFunction` (Prend la valeur existante et la nouvelle valeur et renvoie la nouvelle valeur)|`BiFunction` (Prend la valeur existante et la nouvelle valeur et renvoie la nouvelle valeur)|Function (Prend une clé et renvoie la nouvelle valeur)|

|La clé a ...|La fonction de mapping retourne ...|`merge`|`computeIfAbsent`|`computeIfPresent`|
|---|---|---|---|---|
|Une valeur `null` dans la map|`null`|Supprime la clé de la map|Ne fait pas de changement dans la map|Ne fait pas de changement dans la map|
|Une valeur `null` dans la map|pas `null`|Met à jour la clé en fonction du résultat de la fonction|Ajoute la clé avec le retour de la fonction comme sa valeur|Ne fait pas de changement dans la map|
|Une valeur non-`null`|`null`|Supprime la clé de la map|Ne fait pas de changement dans la map|Supprime la clé de la map|
|Une valeur non-`null`|pas `null`|Met à jour la clé avec le résultat de la fonction|Ne fait pas de changement dans la map|Met à jour la clé avec le résultat de la fonction|
|La clé n'est pas dans la map|`null`|Ajoute la clé à la map|Ne fait pas de changement dans la map|Ne fait pas de changement dans la map|
|La clé n'est pas dans la map|pas `null`|Ajoute la clé à la map|Ajoute la clé à la map avec le retour de la fonction comme valeur|Ne fait pas de changement dans la map|