## Utiliser les Lists, Sets, Maps and Queues
Une *collection* est un groupe d'objets contenus dans un seul objet.

Le *Java Collections Framework* est un ensemble de classes de `java.util` pour entreposer les collections.

Il y a quatre principales interfaces dans le *Java Collections Framework* :
- `List` : une *liste* est une collection ordonnée d'éléments qui accepte les doublons. Les éléments d'une list sont accessibles par un index `int`,
- `Set` : Un *set* est une collection qui ne supporte pas de doublons,
- `Queue` : une *queue* est une collection qui ordonne les éléments dans un ordre spécifique. Une queue normale est FIFO, mais d'autres configurations sont possibles,
- `Map` : une *map* est une collection qui lie des clés à des valeurs, où les doublons de clés ne sont pas autorisés. Les éléments d'une map sont des paires clé/valeur.

L'interface `Collection` est la racine de toutes les collections sauf map.
```
        Collections                     Map
             |
 --------------------------
 ^           ^            ^  
 |           |            |
List        Set         Queue
``` 
### Méthodes communes des collections
#### `add()`
La méthode `add()` insère un nouvel élément dans la `Collection` et retourne si elle a réussi à le faire.
```java
boolean add(E element)
```
Par exemple, si vous essayez d'insérer un doublon dans un Set, `add()` renverra `false`.
#### `remove()`
La méthode `remove()` supprime un **unique** élément correspondant et retourne si elle a réussi à le faire.
```java
boolean remove(Object object)
```
> Appeler `remove()` avec un `int` utilise l'index. Faire attention quand le type de l'élément de la collection est aussi un int.

Un index inexistant lancera une `IndexOutOfBoundsException`.
#### `isEmpty()` et `size()`
Les méthodes `isEmpty()` et `size()` regardent combien il y a d'éléments dans la `Collection`.
```java
boolean isEmpty()
int size()
```
#### `clear()`
La méthode `clear()` fournit un moyen simple d'éliminer tous les éléments de la `Collection`.
```java
void clear()
```
#### `contains()`
La méthode `contains()` vérifie la présence d'un certain élément dans la `Collection`.
```java
boolean contains(Object object)
```
`contains()` appelle la méthode `equals()` sur tous les éléments.
### Utiliser l'interface `List`
Vous pouvez utiliser une *liste* lorsque vous souhaitez une collection ordonnée pouvant contenir des doublons.

Les objets peuvent être consultés et insérés à des positions spécifiques grâce à un index, d'une manière semblable à celle des *arrays*.

Bien que les classes implémentant l'interface `List` aient beaucoup de méthodes, vous n'avez qu'à connaître les principales pour l'examen. 

La principale similarité des implémentations de `List` est qu'elles sont **ordonnées** et **autorisent les doublons**.
#### Comparaison des implémentations de `List`
##### Comparaison asymptotique (*Big O notation*)
On utilise la comparaison asymptotique pour les performances des algorithmes, c'est la différence d'ordre de grandeur (*order of magnitude difference*).
Comparaisons asymptotiques courantes :
- `O(f)` - *temps constant* - Peu importe la taille de la collection, la réponse mettra toujours le même temps pour arriver,
- `O(log n)` - *temps logarithmique* - un logarithme est une fonction mathématique qui évolue plus lentement que la taille des données (exemple: `binarySearch`),
- `O(n)` - *temps linéaire* - Les performances augmentent linéairement par rapport à la taille de la collection,
- `O(n²)` - *temps quadratique* - code qui contenant des boucles ou chaque boucle passe à travers les données

> Wikipedia : [Complexité en temps](https://fr.wikipedia.org/wiki/Complexit%C3%A9_en_temps) 
##### `ArrayList`
Une `ArrayList` est comme un tableau redimensionnable. 

Quand un élément est ajouté, l'`ArrayList` croît automatiquement.

Quand vous n'êtes pas sûrs de quelle collection utiliser, utilisez `ArrayList`.

###### `LinkedList`
Une `LinkedList` est spéciale car elle implémente `List` et `Queue`.

Elle possède des méthodes pour faciliter l'insertion et la suppression d'éléments en début et/ou fin de la liste.

Le principal avantage de la `LinkedList` est que vous pouvez accéder à des éléments au début et à la fin de la liste en temps constant.

Le compromis est que le traitement d'un indice arbitraire prend un temps linéaire.

###### `Vector`
Historiquement, `Vector` était le choix par défaut si vous vouliez une liste.

Puis `ArrayList` l'a remplacé, puisque `ArrayList` fait la même chose plus vite.

Le seul avantage de `Vector` face à `ArrayList` est qu'il est thread-safe.
###### `Stack`
Une `Stack` est une structure de donnée où vous pouvez ajouter des éléments sur le dessus de la pile (comme une pile d'assiettes).

`Stack` étend `Vector` et n'est plus utilisé à l'heure actuelle.

Si vous avez besoin d'une pile, utilisez `ArrayDeque`
#### Travailler avec les méthodes de `List`
Les méthodes contenues dans l'interface `List` sont faites pour marcher avec des index.

| Méthode|Description|
|---|---|
|`void add(E element)`|Ajoute un élément à la fin|
|`void add(int index, E element)`|Ajoute un élément à l'index indiqué et bouge le reste vers la fin|
|`E get(int index)`|Renvoie l'élément à l'index indiqué |
|`int indexOf(Object o)`|Renvoie le premier index correspondant (ou `-1` si non trouvé)|
|`int lastIndexOf(Object o)`|Renvoie le dernier index correspondant (ou `-1` si non trouvé)|
|`void remove(int index)`|Supprime l'élément à l'index indiqué et bouge le reste vers le début|
|`E set(int index, E e)`|Remplace l'élément à l'index et renvoie l'original|

À l'OCA vous avez appris à parcourir une liste
```java
for (String string: list) {
    System.out.println(string);
}
```
Avant Java 5 il fallait faire :
```java
Iterator iter = list.iterator();
for (iter.hasNext()) {
    String string = (String) iter.next();
    System.out.println(string);
}
```
### Utiliser l'interface `Set`
Vous utilisez un `Set` lorsque vous ne souhaitez pas avoir de doublons.
#### Comparaison des implémentations de `Set`
##### `HashSet`
Un `HashSet` stocke ses éléments dans une [table de hachage](https://fr.wikipedia.org/wiki/Table_de_hachage).

Cela veut dire qu'il utilise la méthode `hashCode()` sur les objets pour les retrouver plus efficacement.

Le principal avantage est qu'ajouter un élément ou vérifier la présence d'un élément dans un set prennent des temps constants.

Le désavantage est que vous perdez l'ordre dans lequel vous avez inséré l'élément.

C'est le set le plus commun.
##### `TreeSet`
Le `Treeset` stocke ses éléments dans une [structure arborescente](https://fr.wikipedia.org/wiki/Arbre_enracin%C3%A9) triée.

Le principal avantage du `TreeSet` est que les set est toujours trié dans l'ordre.

Le désavantage est qu'ajouter un élément ou vérifier la présence d'un élément dans un set prennent des temps logarithmiques (`O(log n)`).

`TreeSet` implémente une interface spéciale, appelée `NavigableSet`, qui vous permet de découper (*slice up*) les collections
#### Travailler avec les méthodes de `Set`
> Pour l'examen, l'interface `Set` n'ajoute pas de méthodes à connaitre autres que celles de `Collection`

La méthode `add()` renvoie true sauf si vous essayez d'insérer un doublon.

> Rappel: La méthode `equals()` est utilisée pour déterminer l'égalité, la méthode `hashCode()` est utilisée pour savoir dans quel « panier » chercher pour que Java n'aie pas à chercher dans tout le set.
#### L'interface `NavigableSet` 
`TreeSet` implémente l'interface `NavigableSet`.

 |Méthode|Description|
 |---|---|
 |`E lower(E e)`|Renvoie le plus grand élément strictement inférieur à e (`< e`), ou `null` if il n'y en a pas|
 |`E floor(E e)`|Renvoie le plus grand élément inférieur ou égal à e (`<= e`), ou `null` if il n'y en a pas|
 |`E ceiling(E e)`|Renvoie le plus grand élément supérieur ou égal à e (`>= e`), ou `null` if il n'y en a pas|
 |`E higher(E e)`|Renvoie le plus grand élément strictement supérieur à e (`>= e`), ou `null` if il n'y en a pas|
 
### Utiliser l'interface `Queue`
On utilise les `Queue` lorsque l'on souhaite que les éléments soient ajoutés et supprimés dans un ordre spécifique.

Les `Queues` sont typiquement utilisées pour trier les éléments avant de les traiter.

Sauf indication contraire, les `Queues` sont présumées *FIFO (First in First Out)*.
#### Comparaison des implémentations de `Queue`
##### `LinkedList`
Vu précédemment (dans les `List`).

En plus d'être une liste, une `LinkedList` est une file d'attente à double entrée.

Le principal avantage de la `LinkedList` est qu'elle implément `List` et `Queue`.

Le compromis est que le traitement n'est pas aussi efficace qu'une « vraie » `Queue`.
##### `ArrayDeque`
Introduit en Java 6, les `ArrayDeque` stockent leurs éléments dans un tableau redimensionnable.

L'avantage des `ArrayDeque` est qu'elles sont plus efficaces que les `LinkedList`.

*Deque* est censé être prononcé « deck ».

##### Travailler avec les méthodes de `Queue`
Sauf pour `push()` toutes sont dans l'interface de `Queue` (C'est ce qui fait qu'`ArrayDeque` est à double entrée)

|Méthode|Description|pour `Queue`|pour `Stack`|
|---|---|---|---|
|`boolean add(E e)`|Ajoute un élément à la fin de la queue. Retourne true ou lance une exception|✔|❌|
|`E element()`|Retourne le prochain élément ou lance une exception si la queue est vide|✔|❌|
|`boolean offer(E e)`|Ajoute un élément à la fin de la queue. Retourne le succès|✔|❌|
|`E remove()`|Supprime et retourne le prochain element ou lance une exception si la queue est vide|✔|❌|
|`void push(E e)`|Ajoute un élément au début de la queue|✔|✔|
|`E poll()`|Supprime et retourne le prochain élément, retourne `null` si vide|✔|❌|
|`E peek()`|Retourne le prochain élément ou retourne `null` si la queue est vide|✔|✔|
|`E pop()`|Supprime et retourne le prochain élément ou lance une exception si la queue est vide|❌|✔|

Communément :
- Pour LIFO (stack) : push / poll / peek
- Pour FIFO (queue à simple entrée) : offer / poll / peek

> Une `LinkedList` marche exactement de la même manière qu'`ArrayDeque`
### `Map`
On utilise une `Map` lorsque l'on souhaite identifier des valeurs grace à des clés.
#### Comparer les implémentations de `Map`
##### `HashMap`
Une `Hahmap` conserve ses clés dans une [table de hachage](https://fr.wikipedia.org/wiki/Table_de_hachage).

Cela veut dire qu'il utilise la méthode `hashCode()` sur les objets pour les retrouver plus efficacement.

Le principal avantage d'ajouter des éléments et les récupérer par leur clé est que ca fonctionne en temps constant.

Le désavantage est que vous perdez l'ordre dans lequel vous avez inséré les éléments. La plupart du temps cela ne vous interesse pas lorsque vous utilisez une `Map`.

###### `TreeMap`
Une `TreeMap` stocke ses éléments dans une [structure arborescente](https://fr.wikipedia.org/wiki/Arbre_enracin%C3%A9) triée.

Le principal avantage est que les clés sont toujours triées dans l'ordre d'insertion.

Le désavantage est qu'ajouter un élément ou vérifier la présence d'une clé prennent des temps logarithmiques (`O(log n)`).
###### `Hashtable`
Tout comme `Vector`, `Hashtable` est une vieille implémentation.

Elle est *thread-safe*.

Le « t » minuscule est une erreur qui a été conservée.
#### Travailler avec les méthodes de `Map`
Comme `Map` n'étend pas `Collection`, il y a beaucoup de méthodes à apprendre.

|Méthode|Description|
|---|---|
|`void clear()`|Supprime toutes les clés et les valeurs de la `Map`.|
|`boolean isEmpty()`|Retourne si la `Map` est vide ou non.|
|`int size()`|Retourne le nombre d'entrées (paires clé/valeur) de la `Map`.|
|`V get(Object key)`|Retourne la valeur correspondante à la `key` ou null si aucune ne correspond.|
|`V put(K key, V value)`|Ajoute ou remplace une paire clé/valeur. Retourne l'ancienne valeur ou `null`.|
|`V remove(Object key)`|Supprime et retourne la valeur correspondante à la clé. retourne `null` s'il n'y en a pas.|
|`boolean containsKey(Object key)`|Retourne si la clé est dans la `Map`.|
|`boolean containsValue(Object value)`|Retourne si la valeur est dans la `Map`.|
|`Set keySet()`|Retourne un set de toutes les clés.|
|`Collection<V> values()`|Retourne une `Collection` de toutes les valeurs.|

> Les `Map` ne peuvent pas utiliser `contains()`,c'est une méthode de `Collection` mais pas de `Map`
### Comparaison des types de `Collection`
|Type|Peut contenir des doublons|Éléments ordonnés|Possède des clés et des valeurs|On doit ajouter/supprimer des éléments dans un ordre spécifique|
|---|---|---|---|---|
|`List`|✔|✔ (par index)|❌|❌|
|`Map`|✔ (pour les valeurs)|❌|✔|❌|
|`Queue`|✔|✔ (récupérés dans un ordre défini)|❌|✔|
|`Set`|❌|❌|❌|❌|

|Type|Interface du *Java Collections Framework*|Trié ?|Appelle `hashCode()`|Appelle `compareTo()`|
|---|---|---|---|---|
|`ArrayList`|`List`|❌|❌|❌|
|`ArrayDeque`|`Queue`|❌|❌|❌|
|`HashMap`|`Map`|❌|✔|❌|
|`HashSet`|`Set`|❌|✔|❌|
|`Hashtable`|`Map`|❌|✔|❌|
|`LinkedList`|`List`, `Queue`|❌|❌|❌|
|`Stack`|`List`|❌|❌|❌|
|`TreeMap`|`Map`|✔|❌|✔|
|`TreeSet`|`Set`|✔|❌|✔|
|`Vector`|`List`|❌|❌|❌|

#### `null` dans les `Collection`
Les structures de données permettant le tri n'acceptent pas les `null`

On ne peut pas insérer de `null` dans une `ArrayDeque` car les méthodes comme `poll()` se servent de `null` comme une valeur de retour spéciale pour indiquer que la `Collection` est vide. Java interdit donc les `null` dans `ArrayDeque`.

`Hashtable` n'accepte pas les `null`, il n'y a pas de raison spéciale, ca a juste été mal fait à la base.

Récapitulatif :
- `TreeMap` - pas de clés `null`,
- `Hashtable` - ni clés ni valeurs `null`,
- `TreeSet` - pas d'élément `null`,
- `ArrayDeque` - pas d'élément `null`.

> L'examen s'attend à ce que vous puissiez choisir la bonne `Collection` pour la situation donnée.