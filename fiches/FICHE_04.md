# Génériques et Collections
Le *Java Collections Framework* inclut les classes qui implémentent `List`, `Map`, `Queue` et `Set`.

## Array et ArrayList

> Rappelez vous que Java compte à partir de 0.

### Array
Un `Array` est une structure de donnée intégrée qui contient d'autres objets ou des primitives.

On peut accéder aux éléments d'un `Array` avec les crochets `[]`.

On peut vérifier le nombre d'éléments d'un `Array` grâce à la variable `length`.

### ArrayList
Une `ArrayList` est un objet qui peut contenir d'autres objets. 

Une `ArrayList` ne peut pas contenir de primitives.

On peut accéder aux éléments d'une `ArrayList` avec `get()`.

On peut vérifier le nombre d'éléments d'une `ArrayList` grâce à `size()`.

### List
Une liste est semblable à un tableau redimensionnable.

Il fait sens de convertir un `array` en `List`. Il ne fait pas sens de convertir un `array` en `Set`. Or cela est possible en procédant en deux temps, On convertit tout d'abord l'`array` en `List` puis la `List` en `Set`.

Les implémentations de `List` peuvent avoir leur comportement propre. 

L'implémentation utilisée lorsque `asList()` est appelé a la fonctionnalité de ne pas être redimensionnable mais d'honorer toutes les autres méthodes de cette interface.

### Recherche et tri
```java
int[] numbers = {6, 9 1, 8};
Arrays.sort(numbers);
System.out.println(Arrays.binarySearch(numbers, 6)); // 1
System.out.println(Arrays.binarySearch(numbers, 6)); // -2
```

```java
List<Integer> list = Arrays.asList(9, 7, 5, 3);
Collections.sort(numbers);
System.out.println(Collections.binarySearch(numbers, 6)); // 0
System.out.println(Collections.binarySearch(numbers, 6)); // -1
```

On doit appeler `sort()` et `binarySearch()` sur `Collections` plutôt que sur `Collection` car `Collection` est une interface et ne possède pas de méthode concrètes.

### Les *wrappers* et l'*autoboxing*
À chaque primitive correspond une classe *wrapper*.

l'*autoboxing* convertit automatiquement une primitive à la classe *wrapper* correspondante si nécessaire si le type générique est spécifié dans la déclaration.

L'*unboxing* convertit automatiquement un wrapper vers la primitive.

|Type primitif|Classe *wrapper*|Exemple d'initialisation|
|---|---|---|
|`boolean`|`Boolean`|`new Boolean(true)`|
|`byte`|`Byte`|`new Byte((byte) 1)`|
|`short`|`Short`|`new Short((short) 1)`|
|`int`|`Integer`|`new Integer(1)`|
|`long`|`Long`|`new Long(1)`|
|`float`|`Float`|`new Float(1.0)`|
|`double`|`Double`|`new Double(1.0)`|
|`char`|`Character`|`new Char('c')`|

On utilise la méthode `get()` pour obtenir un élément d'une liste, `add()` pour en ajouter, `remove()` pour en supprimer.

> Attention : la méthode `remove()` de `List` possède deux surcharges, une prenant l'index de l'élément à supprimer, l'autre prenant l'objet à supprimer (peut être troublant sur une liste d'Integer)

### L'opérateur diamant
```java
List<String> = new ArrayList<>();
```

L'opérateur diamant n'est pas limité aux déclarations d'une seule ligne.
 ```java
// Exemple parfaitement légal
import java.util.*;

class Doggies { 
    List<String> names;
    
    Doggies() {
        names = new ArrayList<>(); // match la variable d'instance
    }
    
    public void copyNames() {
        ArrayList<String> copyNames;
        copyOfNames = new ArrayList<>(); // math la variable locale
    }
}
 ```

### Travailler avec les génériques
Une liste non-générique peut contenir n'importe quoi.

Les génériques règlent ce problème en vous permettant d'écrire et d'utiliser des types paramétrés.

```java
List<String> names = new ArrayList<String>();
names.add(new StringBuilder("Webby"); // <-- NE COMPILE PAS
```

### Classes Génériques
La syntaxe pour introduire un générique est de déclarer le *paramètre de type formel* entre crochets `< >`.

Le nom du type peut être celui que vous voulez, par convention :
- `E` pour un élément,
- `K` pour la clé d'une map,
- `V` pour la valeur d'une map,
- `N` pour un nombre,
- `T` pour un type générique,
- `S`, `U`, `V` et ainsi de suite pour les types génériques suivants.

Les classes génériques sont pratiques quand les classes utilisées comme paramètre n'ont rien avoir les unes avec les autres.

Les classes génériques ne sont pa limitées a un seul type.

eg.
```java
public class SizeLimitedCrate<T, U> { 
    // ...
}
```
En coulisse, le compilateur remplace toutes les références aux génériques par `Object`.

L'effacement de type (*type erasure*) est le fait d'enlever les génériques de votre code. Le compilateur le fait automatiquement. 

Cela assure la rétro-compatibilité avec les versions de Java n'ayant pas les génériques.

### Interfaces génériques
Tout comme les classes, les interfaces peuvent déclarer un type de parametre formel (*formal type parameter*).
```java
public interface Shippable<T> { 
    void ship(T t);
}
```
#### Première façon d'implémenter une interface générique
```java
// Créer un classe concrète
class ShippableRobotCrate implements Shippable<Robot> { 
    public void ship(Robot robot);
}
```
#### Deuxième façon d'implémenter une interface générique
```java
// Créer une classe générique
class ShippableAbstractCrate<U> implements Shippable<U> { 
    public void ship(U t);
}
```
#### Troisième façon d'implémenter une interface générique
```java
// Ne pas utiliser les génériques
class ShippableCrate implements Shippable { 
    public void ship(Object t);
}
```
#### Qu'est-ce qu'on ne peut pas faire avec les génériques ?
> La plupart des limitations sont dues à l'effacement de type (*type erasure*)

- **appeler le constructeur**, il n'est pas permis d'utiliser `new T()` puisqu'à l'exécution il serait converti en `new Object`,
- **créer un tableau de ce type statique**, car vous créeriez un tableau d'`Object`,
- **appeler `instanceof`**. Ce n'est pas permis car à l'exécution `List<Integer>` et `List<String>` seraient les mèmes à cause de l'effacement de type,
- **utiliser un type primitif comme paramètre de type générique**. Ce n'est pas un problème car vous pouvez utiliser les classes wrapper à la place,
- **créer des variables statiques du paramètre de type générique**. Ce n'est pas permis car le type est lié à l'instance de la classe.

### Méthodes génériques
C'est pratique pour les méthodes statiques mais sont aussi autorisées pour les méthodes non-statiques.
```java
// Exemple de méthode générique
public static <T> Crate<T> ship(T t) {
    System.out.println("Preparing" + t);
    return new Crate<T>();
}
```
- le paramètre de la méthode est le type générique `T`,
- le type de retour est `Crate<T>`,
- avant le type de retour, on déclare le type formel `<T>`.

>Ne pas oublier la déclaration du type formel si la classe n'obtient pas le type formel par sa classe ou son interface.
```java
public static <T> void sink(T t) {}
public static <T> T sink(T t) { return t; }
public static T noGood(T t) { return t; } // <-- NE COMPILE PAS
```

#### Syntaxe optionnelle pour l'invocation de méthode générique
Vous pouvez appeler une méthode générique normalemet, et le compilateur trouvera le type que vous voulez.

Autrement, vous pouvez spécifier le type explicitement pour mettre en évidence le type que vous voulez.
```java
Box.<String>ship("package");
Box.<String[]>ship(args);
```
### Interagir avec le code *legacy*
Le code *legacy* est de l'ancien code 

### Bornes
Les génériques ne semblent pas particulièrement utiles puisqu'ils sont considéré comme des `Objects` et de ce fait ont peu de méthodes à disposition. Les types `wildcard` règlent ce problème en limitant quels types peuvent être utilisés à leurs place.

Un paramètre de type borné (*bounded parameter type*) est un type génériques qui d'efinit une limite pour ce générique.

Un type générique wildcard est un type générique inconnu représenté par un `?`.

<table>
    <tr>
        <th>Type de borne</th>
        <th>Syntaxe</th>
        <th>Exemple</th>
    </tr>
    <tr>
        <td>wildcard sans borne</td>
        <td>
            <pre lang="java">
                ?
            </pre>
        </td>
        <td>
            <pre lang="java">
               List&lt;?&gt; l = new ArrayList&lt;String&gt;();        
            </pre>
        </td>
    </tr>
    <tr>
        <td>wildcard avec une borne haute</td>
        <td>
            <pre lang="java">
                ? extends type
            </pre>
        </td>
        <td>
            <pre lang="java">
               List&lt;? extends Exception&gt; l = new ArrayList&lt;RuntimeException&gt;();        
            </pre>
        </td>
    </tr>
    <tr>
        <td>wildcard avec une borne basse</td>
        <td>
            <pre lang="java">
                ? super type
            </pre>
        </td>
        <td>
            <pre lang="java">
               List&lt;? super Exception&gt; l = new ArrayList&lt;Object&gt;();        
            </pre>
        </td>
    </tr>
</table>

### Wildcard sans borne
Un *wildcard* sans borne représente n'importe quel type de données.

On utilise `?` pour dire que n'importe quel type nous va.

### Wildcard avec une borne haute
```java
List<? extends Number> list = new ArrayList<Integer>();
```
la wildcard avec une borne haute dit que `Number` ou n'importe quelle classe qui étend `Number` peut être utilisé comme type de paramètre formel

> Attention, il faut utiliser le mot-clé `extends` et non pas le mot-clé `implements` même si c'est une interface.
### Wildcard avec une borne basse
<table>
    <tr>
        <th>Code</th>
        <th>La méthode compile</th>
        <th>On peut passer List&lt;String&gt;</th>
        <th>On peut passer List&lt;Object&gt;</th>
    </tr>
    <tr>
        <td>
            <pre lang="java">
                public static void addSound(List&lt;?&gt; list) {
                    list.add("quack");
                }        
            </pre>
        </td>
        <td>
            ❌ (Les génériques sans borne sont immutables)
        </td>
        <td>
            ✔  
        </td>
        <td>
            ✔
        </td>
    </tr>
    <tr>
        <td>
           <pre lang="java">
               public static void addSound(List&lt;? extends Object&gt; list) {
                   list.add("quack");
               }        
           </pre>
        </td>
        <td>
           ❌ (Les génériques avec une borne haute sont immutables)
        </td>
        <td>
           ✔  
        </td>
        <td>
           ✔
        </td>
    </tr>    
    <tr>
        <td>
         <pre lang="java">
             public static void addSound(List&lt;Object&gt; list) {
                 list.add("quack");
             }        
         </pre>
        </td>
        <td>
            ✔
        </td>
        <td>
           ❌ (avec les génériques, on doit passer une correspondance exacte)  
        </td>
        <td>
           ✔
        </td>
    </tr>
    <tr>
        <td>
         <pre lang="java">
             public static void addSound(List&lt;? super String&gt; list) {
                 list.add("quack");
             }        
         </pre>
        </td>
        <td>
           ✔
        </td>
        <td>
           ✔
        </td>
        <td>
           ✔
        </td>
    </tr>
</table>

Comme les classes génériques vous n'aurez probablement pas à les utiliser sauf si vous créer du code amené à être utilisés par d'autres personnes.

### Conclusion
En considérant
```java
class A {}
class B extends A {}
class C extends B {}
```
#### ✔ Exemple 1
```java
// Exemple de bonne utilisation des génériques
<T> T method1(List<? extends T> list) {
    return list.get(0);
}
```
#### ❌ Exemple 2
```java
// Exemple de mauvaise utilisation des génériques
// Le type de retour n'est pas un type, vous êtes censés savoir ce que votre méthode retourne
<T> <? extends T> method2(List<? extends T> list) { // <-- NE COMPILE PAS
    return list.get(0);
}
```
#### ❌ Exemple 3
```java
// Exemple de mauvaise utilisation des génériques
// Conflit entre le nom de la classe B et le parametre typé B
<B extends A> B method3(List<B> list) { 
    return new B(); // <-- NE COMPILE PAS
}
```
#### ✔ Exemple 4
```java
// Exemple de bonne utilisation des génériques
void method4(List<? super B> list) { }
```
#### ❌ Exemple 5
```java
// Exemple de mauvaise utilisation des génériques
// Une wildcard doit avoir un ? dans sa déclaration
<X> method5(List<X super B> list) { } // <-- NE COMPILE PAS
```