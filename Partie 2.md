# Patrons de conception et principes
Au mieux votre application est conçue, au mieux elle va s'adapter aux changements de besoins, lui permettant de s'échelonner naturellement au cours de la durée de vie du projet.

## Concevoir une interface
Comme vous vous en souvenez, une interface est un type de données abstrat, similaire à une classe, définissant une liste de méthodes abstraites publiques que les classes implémentant cette interfaces doivent fournir.

Les interfaces peuvent aussi contenir des variables constantes `public static final`, des méthodes `default` et des méthodes `static`.

Une interface peut étendre une autre interface, ce qui fait qu'elle hérite de toutes ses méthodes abstraites.

Les interfaces permettent aussi un support limité de l'héritage multiple en Java, car une classe peut implémenter plusieurs interfaces.

Règles :
- une interface ne peut pas étendre une classe comme une classe ne peut étendre une interface,
- une interface ne peut pas être `final`,
- Java ne compilera pas si une classe ou une interfae hérite de deux méthodes `default` avec la même signature et ne fournit pas sa propre implémentation.

> Si vous avez encore des problèmes avec les interfaces, nous vous suggérons de revenir au chapitre de l'OCA concernant les interfaces

### Objectif des interfaces
Une interface fournit un moyen pour une personne de développer du code utilisant le code d'une autre personne sans avoir accès à son implémentation.

Elles permettent de faciliter le développement d'applications en permettant aux équipes de travailler en parallèle plutôt qu'en étant dépendants les unes des autres.

Un développeur utilisant une interface peut créer un *mock* temporaire, aussi appelé "code fictif" (*dummy code*) qui simule l'objet qui implémente cette interface avec une implémentation simplifiée.

## Introduction à la programmation fonctionnelle
Java définit une interface fonctionnelle (*functional interface*) comme une interface composée uniquement d'une seule méthode abstraite.

Les interfaces fonctionnelles sont une base pour les expressions lambda en programmation fonctionnelle

Une expression lambda est un block de code qui se transmet, comme une méthode anonyme. (~~)

### Définir une interface fonctionnelle
```java
@FunctionalInterface
public interface Sprint {
    public void sprint(Animal animal);
}
```
### @FunctionalInterface
C'est une bonne pratique d'annoter vos interfaces fonctionnelles avec l'annotation `@FunctionalInterface` , même si ce n'est pas requis en programmation fonctionnelle.

Si une classe est annotée avec `@FunctionalInterface` et qu'elle possède plus d'une méthode abstraite, ou aucune, alors ça ne compilera pas.

Cette annotation permet donc de savoir si l'interface peut être utilisée avec une lambda.

### Implémenter une interface fonctionnelle avec les Lambdas
Java se base sur le contexte pour trouver ce que la lambda veut dire.

Les expressions lambda se basent sur la notion d'exécution différée.

### Comprendre la syntaxe des lambdas
```java
a -> a.canHop();

(Animal a) -> { return a.canHop(); }
```
Syntaxe d'une lambda en omettant les parties optionnelles :
- On spécifie un unique paramètre nommé `a`
- L'opérateur *flèche* sépare les paramètres du corps
- Le corps appelle une unique méthode et retourne le résultat de cette méthode 
```
Parameter name
    |       body
    v    ----------  
    a -> a.canHop()
      ^
      |
    arrow
```
Syntaxe d'une lambda en incluant les parties optionnelles :
- On spécifie un unique paramètre nommé `a` et définissons son type comme `Animal` en enveloppant les paramètres d'entrée entre parentheses `()`
- L'opérateur *flèche* sépare les paramètres du corps
- Le corps à une ligne de code ou plus, incluant des accolades `{}`, un point-virgule `;` et une instruction `return`
```
        Parameter name
            |                body
            v              ----------  
    (Animal a) -> { return a.canHop(); }
      ^        ^      ^              ^
      |        |      |______________|
    optional  arrow    Required because 
   parameter              in block
     type
```
Les parenthèses peuvent être omises s'il y a exactement un paramètre d'entrée et que le type n'est pas implicitement défini.

Les expressions qui ont zéro ou plus d'un paramètre devront avoir des parenthèses.

### Repérer les lambdas invalides
Si vous ajoutez des accolades `{}` alors vous devez explicitement terminer chaque instruction par un point-virgule `;`.

Si vous ajoutez des accolades `{}`, vous devez utiliser l'instruction `return` si la méthode de l'interface fonctionnelle que la lambda implémente renvoie une valeur.

A contrario, l'instruction `return` est optionnelle si la méthode renvoie `void`.

Les instructions lambda ne sont pas forcées d'utiliser tous les paramètres d'entrée.
 
Si l'un des paramètres a un type défini alors tous les paramètres doivent avoir leur type défini.

Java ne nous permet de redéclarer une variable locale dans une lambda

### Appliquer l'interface Predicate
```java
package java.util.function;

public interface Predicate<T> {
    public boolean test(T t);
}
```
Le bénéfice d'utiliser l'interface `Predicate` est qu'elle nous permet de ne plus avoir à créer nos propres interfaces fonctionnelles.

## Implémenter le polymorphisme
Le polymorphisme est la capacité d'une unique interface de prendre plusieurs formes.
 
En Java, cela permet à plusieurs types d'objets d'être passés à une unique méthode ou classe.

On peut accéder à un objet Java en utilisant :
- une référence du même type que l'objet,
- une référence qui est une super-classe de cet objet,
- une référence qui définit une interface que cet objet implémente.
 
Une conversion (cast) n'est aps requise si l'objet est réassigné à un super-type ou une interface de cet objet.
 
Si vous utilisez une variable pour vous référer à un objet, alors seulement les méthodes et les variables qui font partie de la référence de la variable peuvent être appelés sans conversion explicite.   
 
### Différencier objet et référence
En Java tous les objets sont accédés par référence.

Il faut s'imaginer que l'objet est l'entité stockée en mémoire, allouée par la JRE.

Peu importe le type de référence que vous avez pour cet objet, l'objet en lui-même ne change pas.

Regles : 
1. Le type de l'objet détermine quelle propriété existent dans la mémoire de l'objet
2. Le type de la référence de l'objet détermine quelles méthodes et variables sont accessibles par le programme Java

### Conversion de référence d'objets
Règles de conversion : 
1. Convertir un objet d'une sous-classe vers une super-classe **ne nécessite pas** de conversion explicite,
2. Convertir un objet d'une super-classe vers une sous-classe **nécessite** une conversion explicite,
3. Le compilateur ne permettra pas de convertir deux objets n'ayant aucun rapport,
4. Même si le code compile, une exception peut être lancée à l'exécution si l'objet converti n'est pas une instance de la classe cible.

La conversion possède des limites, même si deux classes possèdent une hiérarchie en commun ne veut pas dire qu'une instance de l'une des deux peut être convertie vers l'autre.

À l'exécution une `ClassCastException` peut survenir si l'objet étant référencer n'est pas une instance de la classe cible.

## Comprendre les principes de conception
Un *principe de conception* est une idée établie ou une bonne pratique qui facilite la conception logicielle.

Les principes de conception mènent à :
- un code plus logique,
- du code plus facile à comprendre,
- des classes plus facile à réutiliser dans d'autres occasions et applications,
- du code plus facile à maintenir et qui s'adapte plus au changement de besoins de l'application.

### Encapsuler les données
L'encapsulation est l'idée de combiner des champs et des méthodes d'une classe pour qu'uniquement les méthodes agissent sur les données, plutôt que les utilisateurs de cette classe accèdent aux champs directement.

L'encapsulation est communément implémentée grace à des membres d'instance `private` et des méthodes `public` qui permettent de récupérer ou de modifier les données, appelés respectivement *getters* (accesseur) et *setters* (mutateur).

Même si les variables d'instance peuvent être publiques, il est découragé d'en créer dans un milieu professionnel.

L'idée de l'encapsulation vient du fait qu'aucun acteur autre que la classe elle-même puisse accéder à ses données. 

Avec l'encapsulation, une classe peut maintenir certains *invariants* dans ses données internes.

Un *invariant* est une propriété ou une vérité qui est maintenue même quand la donnée est modifiée.

La plupart du temps. les getters et setters offrent un accès quasi-direct aux variables privées.

C'est une bonne pratique d'encapsuler *toutes* les variables d'une classe, même s'il n'a aucune règle sur ces données.

### Créer des JavaBeans
Un *JavaBean* est un principe pour l'encapsulation de données dans un objet. 

<table>
    <tr>
        <th>Règle</th>
        <th>Exemple</th>
    </tr>
    <tr>
        <td>Les propriétés sont privées</td>
        <td>
            <pre lang="java">
                private int age;
            </pre>
        </td>
    </tr>
    <tr>
        <td>Les getters pour les propriétés non-booléennes commencent par `get`</td>
        <td>
            <pre lang="java">
                public int getAge() { 
                    return age;
                }
            </pre>
        </td>
    </tr>
    <tr>
        <td>Les getters pour le propriétés booléennes peuvent commencer par is ou par get</td>
        <td>
            <pre lang="java">
                public boolean isBird() {
                    return bird;
                }
                public boolean getBird() {
                    return bird;
                }
            </pre>
        </td>
    </tr>
    <tr>
        <td>Les setters commencent par `set`</td>
        <td>
            <pre lang="java">
                public void setAge(int age) {
                    this.age = age;
                }
            </pre>
        </td>
    </tr>
    <tr>
        <td>Les noms de méthode doivent être préfixés par `set` / `get` / `is` suivi de la première lettre en majuscule suivi du reste du nom de la propriété</td>
        <td>
            <pre lang="java">
                public void setNumChildren(int numChildren) {
                    this.numChildren = numChildren;
                }
            </pre>
        </td>
    </tr>
</table>

Même si les méthodes d'accès aux valeurs booléennes peuvent commencer par `is`, le wrapper `Boolean` est exclu de cette règle, elles doivent donc commencer par `get`

### Appliquer la relation ***Is-a*** (est-un)
On décrit la propriété d'un objet à être une instance d'un type de donnée comme une relation *is-a*.

La relation *is-a* est aussi appelé "test d'héritage" (*inheritance test*).

### Appliquer la relation ***has-a*** (possède-un)
La relation *has-a* est la propriété d'un objet à posséder un objet nommé ou une primitive comme l'un de ses membres.

La relation *has-a* est aussi appelé "test de composition de l'objet" *object composition test*.

Les membre privés ne sont pas hérités en Java.

> Des fois, un objet semble passer le *is-a test* mais échoue lorsqu'il est combiné au *has-a test*

### Composer des objets
La composition d'objet est la propriété de construire une classe en utilisant des références à d'autres classes pour pouvoir réutiliser certaines fonctionnalités de celle-ci.

La composition d'objet doit être vu comme une alternative à l'héritage et est souvent utilisée pour simuler un comportement polymorphique qui ne peut être accompli par un simple héritage.

L'avantage de la composition d'ojet est qu'elle à tendance à promouvoir la réutilisabilité de l'objet. En utilisant celle-ci vous obtenez un accès a d'autres classes difficiles a atteindre via l'héritage simple de Java.

La composition d'objet vous obligera à exposer explicitement les méthodes et valeurs sous-jacentes manuellement.

> La composition d'objet et l'héritage ont tout les deux leur place dans le développement de bon code. 