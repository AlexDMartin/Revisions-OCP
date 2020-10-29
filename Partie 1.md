# Advanced Class Design
## Modificateurs d'accès
|  Peut accéder 	|   `private`	| modificateur par défaut (package private)  	|  `protected` 	|   `public`	|
|---	|:---:	|:---:	|:---:	|:---:	|
| Membres dans la même classe  	|   ✔	|   ✔|   ✔	|  ✔	|
| Membres dans une autre classe dans un autre package  	| ❌	|   ✔	|   ✔	|   ✔	|
| Membre d'une super-classe dans un autre package  	| ❌	|  ❌	|   ✔	|   ✔|
| Méthode / champ dans une classe (qui n'est pas une superclasse) dans un autre package  	| ❌ |  ❌|   ❌|  ✔

Le mot clé "*default*" a été ajouté en Java 8 pour les interfaces mais n'est pas un modificateur d'accès.

## Surcharge et redéfinition
La redéfinition (*overriding*) et la surcharge (*overloading*) d'une méthode ne s'appliquent uniquement si le nom de la méthode est le même
- La redéfinition s'applique quand la signature de la méthode (nom de la méthode + liste de paramètres) est la même
- Lors d'une surcharge, les paramètres de la méthode doivent varier en type et/ou en nombre

Quand plusieurs méthodes surchargées sont présentes, Java cherche la correspondance la plus proche en premier.
L'ordre de recherche est le suivant :
- Correspondance exacte par type,
- Correspond au type d'une superclasse,
- Conversion vers un type primitif plus large,
- Conversion vers un type autoboxé,
- varargs.

Pour la redéfinition (override), les méthodes redéfinies doivent se plier à certaines règles :
 - Le modificateur d'accès doit être le même ou plus accessible,
 - Le type de retour doit être le même ou plus restrictif, dît "type de retour covariant"
 - Si une exception checkée est propagée (thrown), alors uniquement cette exception, ou une sous-classe de cette exception est autorisée à être propagée
 
 La méthode ne doit pas être statique 
 &#8594; si c'est le cas, la méthode et cachée (hidden) et non pas redéfinie (overridden)
 
## Classes abstraites 
Une classe abstraite peut potentiellement n'avoir aucune méthode, et n'avoir uniquement des méthodes abstraites.
```java
// Méthode abstraite
abstract void clean();
```
```java
// Implémentation par défaut
void clean() {}
```

Une classe abstraite peut avoir n'importe quel nombre de méthodes, zéro inclus &#8594; Ces méthodes peuvent être abstraites ou concrètes
Des méthodes abstraites ne peuvent apparaitre dans des classes n'étant pas abstraites.
La première sous-classe concrète d'une classe abstraite dois obligatoirement implémenter toutes les méthodes n'étant pas implémentées par une super-classe.
## Static et Final
Le mot-clé *final* empêche une variable de changer ou une méthode d'être redéfinie

Le mot-clé *static* fait qu'une variable est partagée au niveau de la classe et utilise le nom de la classe pour se référer à la méthode.

Les mot-clés *static* et *final* peuvent aussi être trouvés au niveau de la classe : 
- classes statiques (voir partie dédiée),
- utiliser final sur une classe veut dire qu'elle ne peut pas avoir de sous-classe.

Comme pour les méthodes, une classe ne peut pas être *static* **et** *final* (exemple : String est *final*)
## Imports
```java
// Imports statiques
import static java.util.Collections.sort;
// ou
import static java.util.Collections.*;
```
```java
// Imports normaux
import java.util.List;
import java.util.ArrayList;
// ou
import java.util.*;
```
## Utilisation de *instance of*
Dans l'expression `a instanceof b`, l'expression retourne *true* si la référence vers laquelle `a` pointe est :
- une instance de la classe `b`
- une sous-classe de `b` (directe ou indirecte)
- une classe qui implémente l'interface `b` (directement ou indirectement)

Toutes les classes de Java héritent de la classe `Object`.

`x instance of b` est `true` la plupart du temps.

Sauf dans un seul cas, si le littéral *null* ou une référence de variable pointant vers *null* est utilisée avec *instance of*, le résultat est `false`.

Le contrôle la compilation n'est fait que lorsque `instanceof` est appelé sur une classe.

Lorsque l'on utilise `instanceof` pour vérifier qu'un objet est une instance d'une interface, Java attend l'execution pour ajouter ce contrôle.

La raison est qu'une sous-classe pourrait implémenter cette interface sans que le compilateur le sache.

L'opérateur `instanceof` est communément utilisé pour vérifier si une instance est une sous-classe d'un objet particulier avant de faire une conversion (*cast*) explicite.

## Comprendre l'invocation de méthode virtuelles 
Les méthodes virtuelles en Java sont toutes les méthodes non-statiques

L'invocation de méthodes virtuelles veut dire que Java va rechercher dans la sous-classe lors de sa recherche de méthode à appeler/
## Annoter les méthodes redéfinies (*@Override*)
Le code commençant par un `@` est une annotation
- ce sont des informations en plus sur le programme ,
- c'est un type de méta-donnée,
- elles peuvent être utilisées par le compilateur ou même à l'exécution.

L'annotation `@Override` est utilisée pour montrer que vous, le développeur, souhaitez que cette méthode redéfinisse une méthode dans une super-classe ou en implémente une d'une interface. 

Si vous faites une surcharge au lieu d'une redéfinition avec l'annotation `@Override`, vous aurez une erreur à la compilation. 

L'annotation `@Override` n'est autorisée uniquement lors du référencement d'une méthode.

Si vous voyez l'annotation `@Override`, cela veut dire la méthode fait l'une de ces trois choses :
- elle implémente une méthode d'une interface,
- elle redéfinit une méthode d'une super-classe,
- elle redéfinit une méthode déclarée dans `Object`, telle que `hashCode`, `equals` ou `toString`.

## Coder Equals, hashCode et toString
Toutes les classes en Java héritent de `java.lang.Object` directement ou indirectement.
### toString
Fournir des messages (*output*) humainement lisible permet de rendre les choses plus faciles pour les développeurs travaillant sur votre code.

```java
@Override
public String toString() {
    // implémentation
} 
``` 
## equals
Java utilise `==` pour comparer des primitifs ou pour vérifier que deux variables se réfèrent au même objet.

Vérifier si deux objets sont équivalents se fait grace à la méthode `equals()`

>`String` possède une méthode `equals()` qui vérifie si la valeur est la même alors que `StringBuilder` utilise l'implémentation d'`equals()` fournie par `Object` qui vérifie donc que ces deux objets sont les mêmes 
```java
@Override
public boolean equals(Object obj) {
    if(!obj instanceof Lion) return false;
    Lion otherLion = (Lion) obj;
    return this.idNumber == otherLion.idNumber;
}
``` 
Comme `equals()` est une méthode importante, Java fournit un certain nombre de règles dans le contrat de la méthode :
- elle est **reflexive** : pour toute valeur de référence `x` non-nulle : `x.equals(x)` doit retourner `true`,
- elle est **symétrique** : pour toutes valeurs de référence `x` et `y` : `x.equals(y)` ne doit retourner `true` si et seulement si `y.equals(x)` retourne `true`,
- elle est **transitive** : pour toutes valeurs de référence `x`, `y` et `z` : si `x.equals(y)` retourne `true` et que `y.equals(z)` retourne `true`, alors `x.equals(z)` doit retourne `true`,
- elle est **cohérente** (*consistent*) : pour toutes valeurs de réference non-nulles `x` et `y` : Pour chaque appel de `x.equals(y)` elle doit constamment retourner `true` ou constamment retourner `false`, sous réserve qu'aucune information ne change entre chaque appel,
- pour tout valeur de référence `x` non-nulle : `x.equals(null)` doit retourner `false`.

### hashCode 
Un hashcode est un nombre qui met l'instance d'une classe dans un nombre finit de catégories.

Contrat officiel de la Javadoc pour hashCode :
- Dans un même programme, le résultat d'`hashCode()` ne doit pas changer,
- Si `equals()` renvoit `true` en étant appelé sur deux objets, alors appeler `hashCode()` sur chacun de ces objets doit renvoyer la même valeur,
- Si `equals()` renvoit `false` en étant appelé sur deux objets, alors appeler `hashCode()` ne doit **pas forcement** renvoyer la même valeur.

```java
public int hashCode() {
    return keyField + 7 * otherKeyField.hashCode();
}
```
Il est commun de multiplier par un nombre premier lors de la combinaison de plusieurs champs pour le hash.
## Travailler avec les Enums
Une énumération est comme un ensemble fix de constantes

En Java, une `enum` est un classe qui représente une énumération/

Elle fournissent un contrôle de sécurité de type

> Une `enum` est un type, pas un `int`
> On ne peut pas étendre une enum

### Utiliser les enums dans l'instruction switch
Java considère le type `enum` comme implicite.

On ne peut pas comparer un `int` avec une `enum`
### Ajouter des constructeurs, des champs et des méthodes
> Le point-virgule à la fin de l'`enum` est optionnel si et seulement si elle ne contient uniquement une liste de valeurs

```java
public enum Season {
    WINTER("low"), SPRING("medium"), SUMMER("high"), FALL("medium"); // ne pas oublier le point-virgule

    private String expectedVisitors; 
    
    private Season(String expectedVisitors) {
        this.expectedVisitors = expectedVisitors;
    }

    public void printExpectedVisitors() {
        System.out.println(expectedVisitors);
    }
}
```
Pour appeler la méthode `printExpectedVisitors()`:
```java
Season.SUMMER.printExpectedVisitors();
```
Le constructeur est privé car il ne peut être appelé que par l'enum. Le code ne compilerait pas avec avec un constructeur publique.

La première fois que l'on demande une valeur de l'enum, Java construit toutes les valeurs de l'enum
> Après ça, Java retourne les valeurs déjà générées
```java
public enum Season2 {
    WINTER {
        public void printHours() { System.out.println("9am-3pm") ;}
    }, SPRING {
        public void printHours() { System.out.println("9am-5pm") ;}
    }, SUMMER {
        public void printHours() { System.out.println("9am-7pm") ;}
    }, FALL {
        public void printHours() { System.out.println("9am-5pm") ;}
    }; // ne pas oublier le point-virgule

   public abstract void printHours();
}
```
L'enum possède une méthode abstraite. Chaque valeur de l'enum doit implémenter cette méthode.

Si on en oublie une, on a une erreur de compilation.

Si nous ne voulons pas que chaque valeur de l'enum possède une méthode on peut créer une implémentation par défaut et redéfinir (override) les cas spécifiques.
```java
public enum Season3 {
    WINTER {
        public void printHours() { System.out.println("short hours") ;}
    }, SUMMER {
        public void printHours() { System.out.println("long hours") ;}
    }, SPRING, FALL ; // ne pas oublier le point-virgule

   public void printHours() {
        System.out.println("default hours");
   }
}
```
Juste parce qu'une enum peut avoir beaucoup de méthodes ne veut pas dire qu'elle le doit, essayez de garder vos enums simples.
## Créer des *nested classes*
Une *nested class* est une classe définie dans une autre classe.

Une *nested class* qui n'est pas statique est appelée une *inner class*.

Il y a 4 types de *nested classes* :
- une ***member inner class*** est une classe définie au même niveau que les variables d'instance,
- une ***local inner class*** est une classe définie dans une méthode,
- une ***anonymous inner class*** est un cas spécial de *local inner class* qui n'a pas de nom,
- une ***static nested class*** est une classe statique qui est définie au même niveau que les variables statiques.

Elles permettent : 
- d'encapsuler les classes auxiliaires en les limitant aux classes les contenant,
- de créer facilement des classes qui ne seront utilisées qu'à un seul endroit,
- Elle peuvent rendre le code plus facile à lire, sauf si elles sont utilisées à mauvais escient.

### Member inner classes
Une *member inner class* est une classe définie au niveau des membres de la classe (même niveau que les méthodes, les variables d'instance et les constructeurs)

Propriétés :
- elles peuvent être `public`, `private`, `protected`, ou avoir l'accès par défaut (*package private*),
- elles peuvent étendre des classe et implémenter des interfaces,
- elles peuvent être `abstract` ou `final`,
- elles ne peuvent pas déclarer de champs statiques ni de méthodes statiques,
- elles peuvent accéder aux membres de la classe extérieure, même les `private`.

Il n'y a rien de spécial à faire pour accéder à la classe extérieure. 

Une déclaration de *member inner class* ressemble à une déclaration de classe autonome sauf qu'elle est située à l'intérieur d'une autre classe.

Elles peuvent accéder aux variables `private` car, théoriquement, elles sont dans la même classe.

Comme une *member inner class* n'est pas statique, elle nécessitent une instance de la classe extérieure pour être utilisée.
```java
Outer outer = new Outer();
Inner inner = outer.new Inner(); // <--  Création de l'inner class
inner.go()
```

Quand les classes interieures ont les mêmes noms variables que les classes extérieures, on peut les appeler differement en utilisant `this`.

> Les inner interfaces privées sont possibles, elles ne peuvent être accédées que dans la classe dans laquelle elles sont définies. 
> Par contre leurs méthodes sont forcement publiques, l'inner class implémentant cette interface doit aussi la déclarer publique (règles des interfaces + règles de l'override).

### Local inner classes
Une *local inner class* est une *nested class* définie dans une méthode.

Comme les variables locales, les *local inner classes* n'existent pas tant que la méthode n'est pas invoquée.

Elles deviennent hors de portée lorsque la méthode retourne.

On ne peut créer des instances qu'à l'intérieur de la méthode. Ces instances peuvent quand même être retournées (même fonctionnement que les variables locales).

Propriétés :
- elles n'ont pas de modificateur d'accès,
- elles ne peuvent pas être déclarées comme statique et ne peuvent pas déclarer de champs statiques ni de méthodes statiques,
- elles ont accès à tous les champs et toutes les méthodes de la classe l'entourant,
- elles n'ont pas accès aux variables locales d'une méthode à moins que ces variables soient *finales* ou *effectively final*.

*Effectively final* : si le code compile toujours en ajoutant le mot-clé final avant la variable locale, alors elle est effectively final (concept créé en Java 8).

### Anonymous inner classes
Une *anonymous inner class* est une *local inner class* qui n'a pas de nom. 

Elle est déclarée en une seul instruction en utilisant le mot-clé `new`.

Les *anonymous inner classes* sont obligées d'étendre une classe existante ou d'implémenter une interface existante.

Elles sont utiles lorsque vous avez une brève implémentation qui ne sera utilisée nulle part ailleurs.

La définition d'une *anonymous inner class* est la même si vous implémentez une interface ou si vous étendez une classe, java trouve automatiquement celui que vous voulez.

Vous pouvez les déclarer dès que vous en avez besoin, même en paramètre d'une autre méthode.

Elles sont contraires a certains concepts fondamentaux comme la réutilisabilité des classes et la haute cohésion, faites attention à ce qu'elle fasse sens lorsque vous les utilisez.

### Static nested classes
Ce n'est pas une *inner class*.

Une *static nested class* est une classe statique définie au niveau des membres.

Elles peuvent être instanciées sa ns instance de la classe l'entourant, elle ne peut donc pas accéder aux variables d'instance sans un objet explicite de la classe l'entourant.

Ce sont des classes normales **sauf** :
- que l'imbrication crée un nouveau namespace car la classe l'entourant doit être utilisée pour l'appeler,
- elles peuvent être `public`, `private`, `protected`, ou avoir l'accès par défaut (*package private*),
- la classe l'entourant peut accéder à ses champs et ses méthodes.

```java
package bird;

public class Toucan {
    public static class Beak {}
}
```

On peut l'importer en utilisant un import normal
```java
import bird.Toucan.Beak;
```

ou un import statique
```java
import static bird.Toucan.Beak;
```
### Récapitulatif sur les nested classes
|   	|  Member inner classes 	|   Local inner classes	|   Anonymous inner classes	|   Static nested classes 	|
|---	|:---:	|:---:	|:---:	|:---:	|
| Modificateurs d'accès autorisés  	|`public`, `protected`, `private` ou modificateur par defaut (*package private*)|Non, elle est déjà locale à la méthode|Non elle est déjà locale à l'instruction|`public`, `protected`, `private` ou modificateur par defaut (*package private*)|
| Peut étendre des classes et n'importe quel nombre d'interfaces  	|✔|✔|❌ Elle doit avoir exactement une super-classe ou une interface|✔|
| Peut être `abstract`  	|✔|✔|N/A car elle n'a pas de définition de classe|✔|
| Peut être `final`  	|✔|✔|N/A car elle n'a pas de définition de classe|✔|
| Peut accéder aux membre d'instance de la classe l'entourant  	|✔|✔|✔|❌ (pas directement, elle nécessite une instance de la classe l'entourant)|
| Peut accéder aux variables locales de la classe l'entourant  	|❌|✔ si *final* ou *effectively final*|✔ si *final* ou *effectively final*|❌|
| Peut déclarer des methodes statiques  	|❌|❌|❌|✔|