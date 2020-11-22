# Exceptions et assertions
## Étude des exceptions
### Terminologie des exceptions
Une exception est la manière dont Java nous dit : « J'abandonne, je ne sais pas quoi faire dans cette situation, à toi de gérer le problème ».

Lorsque vous créez une méthode, vous pouvez soit gérer l'exception soit en faire le problème du code l'appelant.

Le *happy path* est quand tout se passe comme prévu. Avec du mauvais code, il peut ne pas y avoir d'*happy path*.

### Catégories d'exceptions
Une *runtime exception* (exception à l'exécution) ou exception *unchecked* (*unchecked exception*) peut être captée (*caught*), mais n'a pas l'obligation de l'être.

Une exception *checked* peut être n'importe quelle classe qui étend `Exception` mais n'est pas une *runtime exception*.

Les exceptions *checked* suivent la règle du *handle or declare* (gérer ou déclarer) pour qu'elles soient soit captées soit lancées a l'appelant.

Une erreur est fatale et ne devrait pas être captée dans le programme. Bien que ce soit légal, ce n'est pas une bonne pratique.

```
              java.lang.object
                    ^
                    |
        --> java.lang.Throwable <---
        |                          |
        |                          |
java.lang.Exception         java.lang.Error
        ^
        |
java.lang.RuntimeException
```

|Type|Comment la reconnaître ?|Il est recommandé de la capter ?|Le programme est il forcé de la capter ou de la déclarer ?|
|---|:---:|:---:|:---:|
|Runtime Exception|`RuntimeException` ou l'une de ses sous-classes|✔|❌|
|Checked Exception|`Exception` ou l'une de ses sous classes mais pas `RuntimeException` ni l'une de ses sous-classes|✔|✔|
|Error|`Error` ou l'une de ses sous-classes|❌|❌|
### Exceptions à l'OCP
##### Rappel de l'OCA

|Exception|Utilisée quand ...|Checked ou unchecked ?|
|---|:---:|:---:|
|`ArithmeticException`|Tentative de division par zéro|Unchecked|
|`ArrayIndexOutOfBoundsException`|Tentative d'accès à index d'*array* illégal|Unchecked|
|`ClassCastException`|Tentative de *cast* d'un objet en une sous-classe dont il n'est pas l'instance|Unchecked|
|`IllegalArgumentException`|Tentative d'appel à une méthode avec des paramètres illégaux ou inappropriés|Unchecked|
|`NullPointerException`|Une référence à null est passée à la place d'un objet|Unchecked|
|`NumberFormatException`|Tentative de conversion d'une *String* en type numérique mais la chaîne n'a pas un format approprié|Unchecked|

##### Exceptions checkées de l'OCP
<table>
<thead>
  <tr>
    <th>Exception</th>
    <th>Utilisée quand ...</th>
    <th>Checked ou unchecked ?</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td><code>java.text.ParseException</code></td>
    <td>Tentative de conversion d'une <code>String</code> en nombre<br></td>
    <td>Checked</td>
  </tr>
  <tr>
    <td><code>java.io.IOException</code></td>
    <td rowspan="3">Règle les problèmes avec IO et NIO.2<br><code>IOException</code> est la classe mère. Il y a plusieurs sous-classes.<br>Vous pouvez supposer que toutes les exceptions de <code>java.io</code> sont <i>checked</i><br></td>
    <td rowspan="3">Checked</td>
  </tr>
  <tr>
    <td><code>java.io.FileNotFoundException</code></td>
  </tr>
  <tr>
    <td><code>java.io.NotSerializableException</code></td>
  </tr>
  <tr>
    <td><code>java.sql.SQLException</code></td>
    <td>Règle les problèmes avec les bases de données.<br><code>SQLException</code> est la classe mère.<br>Vous pouvez supposer que toutes les exceptions de <code>java.sql</code> sont <i>checked</i><br></td>
    <td>Checked</td>
  </tr>
</tbody>
</table>

##### *Runtime exceptions* de l'OCP

<table>
<thead>
  <tr>
    <th>Exception</th>
    <th>Utilisée quand ...</th>
    <th>Checked ou unchecked ?</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td><code>java.lang.ArrayStoreException</code></td>
    <td>Tentative de stockage de mauvais type de donnée dans un <i>array</i></td>
    <td>Unchecked</td>
  </tr>
  <tr>
    <td><code>java.time.DateTimeException</code></td>
    <td>Reception d'un format invalide de chaîne de caractère pour une date</td>
    <td>Unchecked</td>
  </tr>
  <tr>
    <td><code>java.util.MissingResourceException</code></td>
    <td>Tentative d'accès à une clé ou à un <i>resource bundle</i> qui n'existe pas</td>
    <td>Unchecked</td>
  </tr>
  <tr>
    <td><code>java.lang.IllegalStateException</code></td>
    <td rowspan="2">Tentative d'exécution d'une opération de concurrence et de collection</td>
    <td rowspan="2">Unchecked</td>
  </tr>
  <tr>
    <td><code>java.lang.UnsupportedOperationException</code></td>
  </tr>
</tbody>
</table>

### La déclaration `Try`
La déclaration `try` consiste en une clause obligatoire `try`. Elle peut avoir une ou plusieurs clause `catch` pour capter les exceptions qui sont lancées (*thrown*). Elle peut aussi avoir une clause `finally` qui est exécutée sans tenir compte du fait qu'une exception a été lancée ou pas.

Ces règles s'appliquent aussi pour le *try-with-resources*.

#### Syntaxe de la déclaration `try`
Un bloc finally ne peut apparaître uniquement dans une déclaration `try`.

Le bloc finally s'execute quoi qu'il arrive, peu importe si une exception a été levée dans le bloc `try`.
```java
try {
    // code protégé
} catch(exceptionType identifier) {
    // gestion de l'exception ( exception handler)
} finally {
    // bloc finally
}
```

Il y a une règle qui indique qu'une déclaration `try` doit posséder soit un bloc `catch` soit un bloc `finally` soit les deux. C'est vrai pour la déclaration `try` mais pas pour la déclaration `try-with-resources`.

- Java vérifie les blocks `catch` dans l'ordre dans lequel ils apparaissent. C'est illégal de déclarer une sous-classe de l'exception dans un bloc `catch` plus bas dans la liste qu'une super-classe car ce serait du code impossible à atteindre (*unreachable code*),
- Java ne vous permettra pas de déclarer un bloc `catch` pour une exception *checked* qui ne peut pas être potentiellement levée par le corps du bloc `try` (*unreachable code*).

### `Throw` vs `Throws`
```java
public String getDataFromDatabase() throws SQLException {
    throw new UnsupportedOperationException();
}
```

> À l'examen, faites attention à ce que `throw` et `throws` ne soient pas inversés.

## Créer des exceptions personnalisées
Lorsque vous créez votre propre exception, vous devez choisir si ce sera une exception *checked* ou *unchecked*.

Bien que vous puissiez étendre n'importe quelle exception, il est d'usage d'étendre `Exception` (*checked*) ou `RuntimeException` (*unchecked*).

```java
// Exemple de création d'exceptions personnalisées
class CannotSwimException extends Exception {}          // checked
class DangerInTheWater extends RuntimeException {}      // unchecked
class SharkInWaterException extends DangerInTheWater {} // unchecked - herité indirectement

class Dolphin {
    public void swim() throws CannotSwimException {
        // logique ici
    }
}
```
### Constructeurs les plus utilisés de la classe Exception
```java
public class CannotSwimException extends Exception {
    
    public CannotSwimException() {
        super();        // Défaut - sans paramètre 
    }

    public CannotSwimException(Exception e) {
        super(e);       // Enveloppe une autre exception
    }

    public CannotSwimException(String message) {
        super(message); // Message personnalisé
    }
}
```
### Affichage 
#### Constructeur par défaut
```java
public static void main(String[] args) {
    throw new CannotSwimException();
}
```
```
Exception in thread "main" CannotSwimException
    at CannotSwimException.main(CannotSwimException.java:18)
```
#### Constructeur avec exception *wrapped* 
```java
public static void main(String[] args) {
    throw new CannotSwimException(new RuntimeException());
}
```
```
Exception in thread "main" CannotSwimException: java.lang.RuntimeException
    at CannotSwimException.main(CannotSwimException.java:19)
Caused by: java.lang.RuntimeException
```
#### Constructeur avec message personnalisé
```java
public static void main(String[] args) {
    throw new CannotSwimException("broke fin");
}
```
```
Exception in thread "main" CannotSwimException: broken fin
    at CannotSwimException.main(CannotSwimException.java:20)
```
## Utiliser le multi-catch
Lorsque quelque chose va mal dans le programme, il est commun de logger l'erreur et de la convertir en un autre type d'exception.
### Exemple - sans multi-catch
```java
// Exemple de conversion - avec duplication de code
public static void main(String[] args) {
    try {
        Path path = Paths.get("dolphinBorn.txt");
        String text = new String(Files.readAllBytes(path));
        LocalDate date = LocalDate.parse(text);
        System.out.println(date);
    } catch (DateTimeParseException e) {
        e.printStackTrace();
        throw new RuntimeException();
    }  catch (IOException e) {
        e.printStackTrace();
        throw new RuntimeException();
    } 
} 
```

> Comment supprimer cette duplication ?

### Piège - Mauvaise approche 
```java
// Exemple de conversion - mauvaise approche
public static void main(String[] args) {
    try {
        Path path = Paths.get("dolphinBorn.txt");
        String text = new String(Files.readAllBytes(path));
        LocalDate date = LocalDate.parse(text);
        System.out.println(date);
    } catch (Exception e) {     // Mauvaise approche
        e.printStackTrace();
        throw new RuntimeException();
    }
} 
```
### Exemple - méthode utilitaire
```java
// Exemple de conversion - avec méthode utilitaire
public static void main(String[] args) {
    try {
        Path path = Paths.get("dolphinBorn.txt");
        String text = new String(Files.readAllBytes(path));
        LocalDate date = LocalDate.parse(text);
        System.out.println(date);
    } catch (DateTimeParseException e) {
        handleException(e);
    }  catch (IOException e) {
        handleException(e);
    } 
} 

private static void handleException(Exception e) {
    e.printStackTrace();
    throw new RuntimeException();
}
```
Le code dupliqué n'est presque plus là mais il y a toujours deux appels à la méthode utilitaire. Le code est aussi dur à lire.
### Bonne approche - multi-catch
En Java 7, ils ont inclus la capacité a *catch* plusieurs exceptions dans le même bloc `catch`, aussi appelé *multi-catch*.
```java
// Exemple de multi-catch
public static void main(String[] args) {
    try {
        Path path = Paths.get("dolphinBorn.txt");
        String text = new String(Files.readAllBytes(path));
        LocalDate date = LocalDate.parse(text);
        System.out.println(date);
    } catch (DateTimeParseException | IOException e) {
        e.printStackTrace();
        throw new RuntimeException();
    }
} 
```
Il n'y a pas de code dupliqué et le code est facile à lire.
### Syntaxe du multi-catch
```java
try {
    // code protégé
} catch(Exception 1 | Exception 2 e) { // catch l'une ou l'autre de ces exceptions
    // gestion de l'exception
}
```
####  Piège - syntaxe
l'examen essaiera de vous piéger avec une syntaxe invalide.

- Les exceptions peuvent être listées dans n'importe quel ordre,
- le nom de la variable ne doit apparaître qu'une fois et à la fin.
```java
// Exemples de syntaxes
catch(Exception1 e | Exception2 e | Exception3 e)       // ❌ NE COMPILE PAS 
catch(Exception1 e1 | Exception2 e2 | Exception3 e3)    // ❌ NE COMPILE PAS 

catch(Exception1 | Exception2 | Exception3 e)           // ✔
```
#### Piège - redondance
```java
try {
    thow new IOException();
} catch (FileNotFoundException | IOException e) {} // ❌ NE COMPILE PAS 
```
`FileNotFoundException` est une sous-classe de `IOException`, le spécifier est redondant.

Le compilateur donnera le message suivant :
```
The exception FileNotFoundException is already caught by the alternative IOException
```
Puisque nous pouvons omettre cette exception sans changer le comportement de notre programme, Java ne le permet pas.

Le code correct est le suivant :
```java
try {
    thow new IOException();
} catch (IOException e) {}
```
#### Piège - le multi-catch est effectively final
Même si c'est une mauvaise pratique, redéfinir une variable dans un bloc catch est légal dans une déclaration `try`.
```java
// Exemple de redéfinition de variable dans une declaration try
try {
    // fais quelquechose
} catch (RuntimeException e) {
    e = new RuntimeException();
}
```
Or ce n'est pas le cas dans un multi-catch :
```java
// eEemple de redéfinition de variable dans un multi-catch
try {
    throw new IOException();
} catch (IOException | RuntimeException e) {
    e = new RuntimeException(); // ❌ NE COMPILE PAS 
}
```
Avec les multi-catchs, nous n'avons plus un type spécifique pour l'exception. En interne, Java utilise la super-classe `Exception` pour la variable.

Pour éviter des problèmes de complexité, Java interdit de pouvoir réassigner la variable de l'exception.

#### Résumé des multi-catchs
```java
public void doesNotCompile() { // ❌ LA MÉTHODE NE COMPILE PAS 
    try {
        mightThrow();
    } catch(FileNotFoundException | IllegalStateException e) {      
    } catch(InputMismatchException e | IllegalStateException e) {   // ❌ Syntaxe - nom de variable en trop
    } catch(SQLException | ArrayIndexOutOfBoundsException e) {      // ❌ Potentiel - on ne peut pas catch SQLException car elle ne peut pas être levée dans le try
    } catch(FileNotFoundException | IllegalArgumentException e) {   // ❌ FileNotFoundException est déjà catch plus haut
    } catch(Exception e) {                                          // ❌ Position - devrait être à la ligne en dessous 
    } catch(IOException e) {                                        // ❌ Position - devrait être à la ligne au dessus 
    }
}
```
## Utiliser le *try-with-resources*
Le multi-catch vous permet d'écrire du code sans duplication.

Un autre problème est la duplication dans le bloc finally.

```java
// Exemple de finally avant Java 7
public void oldApprocach(Path path1, Path path2) throws IOException {
    BufferReader in = null;
    BufferReader out = null;

    try {
        in = Files.newBufferedReader(path1);    
        out = Files.newBufferedReader(path2);
        out.write(in.readLine());    
    } finally {
        if(in!= null) in.close();
        if(out!= null) out.close();
    }
}
```
C'est beaucoup de code pour pas grand-chose et on ne s'occupe même pas des exceptions ...

En java 7 ils ont rajouté la syntaxe *try-with-resources* :
```java
public void newApproach(Path path1, Path path2) throws IOException {
    try(BufferedReader in = Files.newBufferedReader(path1);
        BufferedReader out = Files.newBufferedWriter(path2)) {
        out.write(in.readLine());
    }
}
```
Le nouveau *try-with-resources* ferme automatiquement toutes les ressources ouvertes dans la clause `try`.

Cette fonctionnalité est aussi connue sous le nom d'*automatic resource management*.
### Bases du *try-with-resources*
La clause `finally` est implicite dans le *try-with-resources*

> La déclaration try-with-resources peut omettre le catch et le finally.
> La déclaration try traditionnelle, elle, doit implémenter soit l'un soit l'autre.

```java
// Syntaxe basique
/*
*    N'importe quelle ressource qui doit être clôturée automatiquement 
*          |
*          v
*/  try(BufferedReader r = Files.newBufferedReader(path1);
        BufferedWriter w = Files.newBufferedWriter(path2)) {
        // Code protégé
    }
/*  ^
*   |
* Les ressources sont fermées ici
*/
```
```java
// Syntaxe en incluant les parties optionnelles
    try(BufferedReader r = Files.newBufferedReader(path1);
        BufferedWriter w = Files.newBufferedWriter(path2)) {
        // Code protégé
    } catch (IOException e) {   // Partie optionnelle
        // Gestion de l'exception
    } finally {                 // Partie optionnelle
        // Bloc finally
    }
```
#### Configurations légales
##### Try traditionnel
||**0** bloc `finally`|**1** bloc `finally`|**2** blocs `finally` **ou plus**|
|---|:---:|:---:|:---:|
|**0** bloc `catch`|❌|✔|❌|
|**1** bloc `catch` **ou plus**|✔|✔|❌|
##### Try-with-resources
||**0** bloc `finally`|**1** bloc `finally`|**2** blocs `finally` **ou plus**|
|---|:---:|:---:|:---:|
|**0** bloc `catch`|✔|✔|❌|
|**1** bloc `catch` **ou plus**|✔|✔|❌|

Les ressources créées dans le `try` ne sont dans le scope uniquement dans le bloc `try`
```java
// Exemple scope
try(Scanner s = new Scanner(System.in)) {
    s.nextLine();
} catch(Exception e) {
    s.nextInt(); // ❌ NE COMPILE PAS 
} finally {
    s.nextInt(); // ❌ NE COMPILE PAS 
}
```

### AutoCloseable
Vous ne pouvez pas mettre n'importe quelle classe dans une déclaration *try-with-resource*

```java
public clas Tyrkey {
    public static void main(String[] args) {
        try (Turkey t = new Turkey()) { // ❌ NE COMPILE PAS 
            System.out.println(t);
        }
    }
}
```
Java ne permet pas ça, il ne sait pas comment fermer une Dinde (`Turkey`). Java nous informe avec cette erreur à la compilation :

```
The resource type Turkey does not implement java.lang.AutoCloseable
```
Pour que Java crée une classe dans la clause `try`, elle doit implémenter `AutoCloseable`
```java
// Exemple d'implémentation d'AutoCloseable
public class TurkeyCage implements AutoCloseable {
    public void close() {
        System.out.println("Close gate");
    }

    public static void main(String[] args) {
        try (TurkeyCage t = new TurkeyCage()) {
            System.out.println("put turkeys in");
        }
    }
}
```
L'interface `AutoCloseable` n'a qu'une méthode à implémenter :
```
public void close() throws Exception;
```
Vous pouvez, au choix, lever une Exception, puisqu'elle sera plus spécifique qu'`Exception`
#### Piège - Exception dans `close()`
```java
public class StuckTurkeyCage implements AutoCloseable {
    public void close() throws Exception {
        throw new Exception("Cage door does not close");
    }

    public static void main(String[] args) {
        try (TurkeyCage t = new TurkeyCage()) { // ❌ NE COMPILE PAS 
            System.out.println("put turkeys in");
        }
    }
}
```
La déclaration *try-with-resources* lève une exception *checked*. Comme vous le savez les exceptions *checked* suivent la règle du *handle or declare*.

Le code compilerait si la méthode `main()` déclarait une `Exception`.

Java recommande que la méthode `close()` ne lève pas une `Exception` mais une exception plus spécifique.

Java recommande aussi que la méthode `close()` soit idempotente, ce qui veut dire qu'elle puisse être appelée plusieurs fois sans aucun effet de bord (*side-effect*) ni comportements indésirables lors des exécutions suivantes. Même si ces pratiques sont légales elles sont déconseillées.

#### Bonnes pratiques
```java
class ExampleOne implements AutoCloseable {     // ✔ 
    public void close() throws IllegalStateException { 
        throws new IllegalStateException("Cage door does not close);
    }
}

class ExampleTwo implements AutoCloseable {     // ❌ Devrait lever ue exception plus spécifique
    public void close() throws Exception {
        throws new Exception("Cage door does not close);
    }
}

class ExampleThree implements AutoCloseable {  // ❌ Possède des effets de bord
    static int COUNT = 0;
    public void close() {
        COUNT++;
    }
}
``` 
#### `AutoCloseable` vs `Closeable`
L'interface `AutoCloseable` a été instaurée en Java 7. Avant ç il existait déjà une interface similaire appelée `Closeable`.

Les différences sont :
- `Closeable` restreint le type d'exception levée à `IOException`,
- `Closeable` exige que l'implémentation soit idempotente.

Les créateurs du langage on mit l'accent sur la rétrocompatibilité. `AutoCloseable` est donc moins stricte que `Closeable`. `Closeable` répond donc aux exigences de `AutoCloseable`
### Suppression d'exceptions
Les ressources sont fermées avant que tout bloc `catch` codé par le programmeur soit exécuté.

On peut catch l'exception levée par la méthode `close()` si on le souhaite ou bien on peut laisser le code appelant s'en charger.

Comme les expressions traditionnelles, les exceptions *checked* suivent la règle du *handle or declare*.

Les exceptions *unchecked* n'ont pas à être gérées.

```java
public class JammedTurkeyCage implements AutoCloseable {
    public void close() throws IllegalStateException {
        throw new IllegalStateException("Cage door does not close");
    }

    public static void main(String[] args) {
        try (TurkeyCage t = new TurkeyCage()) { // ❌ NE COMPILE PAS 
            System.out.println("put turkeys in");
        } catch (IllegalStateException e) {
            System.out.println("caught: "+ e.getMessage());
        }
    }
}
```
```
caught: Cage door does not close
```
> Avec une exception *checked*, la déclaration `try` aurait eu à la catcher ou la méthode `main()` aurait eu à la déclarer

#### Accumulation d'exceptions
Java 7 a aussi ajouté une manière d'accumuler les exceptions
```java
try(JammedTurkeyCage t = new JammedTurkeyCage()) {
    throw new IllegalStateException("turkeys ran off");
} catch (IllegalStateException e) {
    System.out.println("caught: " + e.getMessage());
    for (Throwable t: e.getSuppressed()) 
        System.out.println(t.getMessage());
}
``` 
```
caught: turkeys ran off
Cage door does not close
```
#### Exception non gérées
```java
try(JammedTurkeyCage t = new JammedTurkeyCage()) {
    throw new RuntimeException("turkeys ran off");
} catch (IllegalStateException e) {
    System.out.println("caught: " + e.getMessage());
   
}
``` 
Java se souvient quand même des exceptions qui ont été supprimées
```
Exception in thread "main"java.lang.RuntimeException: turkeys ran off
    atJammedTurkeyCage.main(JammedTurkeyCage.java:20)
    Suppressed: java.lang.IllegalStateException: Cage door does not close
    atJammedTurkeyCage.close(JammedTurkeyCage.java:5)
    atJammedTurkeyCage.main(JammedTurkeyCage.java:21)
```
#### Plusieurs exceptions
```java
try(JammedTurkeyCage t1 = new JammedTurkeyCage();
    JammedTurkeyCage t2 = new JammedTurkeyCage()) {
    System.out.println("turkeys entered cages");
} catch (IllegalStateException e) {
    System.out.println("caught: " + e.getMessage());
    for (Throwable t: e.getSuppressed()) 
        System.out.println(t.getMessage());
}
```
Les ressources sont fermées dans l'ordre inverse de leur déclaration. Ici `t2` est fermée en premier puis `t1`, puisque l'exception a déjà été levée elle devient une exception supprimée.
```
turkeys entered cages
caught: Cage door does not close
Cage door does not close
```
#### Piège - Exceptions hors du try
Souvenez-vous que la suppression d'exception ne marche qu'avec les exceptions levées dans la clause `try`.
```java
try(JammedTurkeyCage t = new JammedTurkeyCage()){
    throw new IllegalStateException("turkeys ran off");
} finally {
    throw new RuntimeException("and we couldn't find them");
}
```
l'`IllegalStateException` est levée, Java essaye de clôturer les ressources (ajoute une exception *suppressed*)  puis la `RuntimeException` de `finally` est levée, ce qui fait que l'on perd la première exception.

Java a besoin d'être rétrocompatible, avant Java 7 `try` et `finally` pouvaient lever une exception et `finally` prenait la précédence. Ce comportement doit continuer.
#### Conclusion
- Les ressources sont clôturées à la fin de la clause `try` et avant les clauses `catch` / `finally`,
- Les ressources sont clôturées dans l'ordre inverse de leur création.
```java
public class Auto implements AutoCloseable {
    int num;

    Auto(int num) { this.num = num; }

    public void close() {
        System.out.pintln("Close: " + num);
    }

    public static void main(String[] args) {
        try (Auto a1 = new Auto(1); Auto a2 = new Auto(2) {
            throw new RuntimeException();
        } catch (Exception e) {
            System.out.println("ex");
        } finally {
            System.out.println("finally");
        }  
    }
}
```
```
Close: 2
Close: 1
ex
finally
```
## Relancer des exceptions
C'est un schéma commun de logger puis de relance la même exception.

Supposons :
```java
public void parseData() throws SQLException, DateTimeParseException {}
```
Lorsque nous utilisons cette méthode nous devons gérer ou déclarer (*handle or declare*) les deux exceptions.

On peut utiliser deux catchs avec du code dupliqué ou utiliser le multi-catch.
```java
public void multiCatch() throws SQLException, DateTimeParseException {
    try {
        parseData();
    } catch (SQLException | DateTimeParseException e) { ❌ duplication, les exceptions catchées sont les mêmes que celles déclarées
        System.err.println(e);
        throw e;
    }
}
```
En Java 7 ils ont rendu légal le fait de pouvoir écrire `Exception` dans le bloc `catch`.
```java
public void rethrowing() throws SQLException, DateTimeParseException { 
    try {
        parseData();
    } catch (Exception e) {
        System.err.println(e);
        throw e;
    }
}
```
Les deux exemples précédents sont similaires mais n'ont pas le même comportement (eg. si une `NullPointerException` est levée).
#### Modification - ajout d'exception
Maintenant admettons que nous voulions utiliser le *file system* dans `parseData`.
```java
public void parseData() throws IOException, SQLException, DateTimeParseException {} // ajout d'IOException
```
avec le multi-catch on doit changer :
```java
public void multiCatch() throws IOException, SQLException, DateTimeParseException { // ajout d'IOException
    try {
        parseData();
    } catch (IOException | SQLException | DateTimeParseException e) {               // ajout d'IOException
        System.err.println(e);
        throw e;
    }
}
```
avec Exception on change uniquement la signature :
```java
public void rethrowing() throws IOException, SQLException, DateTimeParseException { // ajout d'IOException
    try {
        parseData();
    } catch (Exception e) {
        System.err.println(e);
        throw e;
    }
}
```
#### Modification - enlever une exception
```java
public void parseData() throws IOException, DateTimeParseException {} // Suppression d'SQLException
```

avec le multi-catch on doit changer :
```java
public void multiCatch() throws IOException, SQLException, DateTimeParseException { // On ne change rien, pour que les appelants n'aient rien à changer
    try {
        parseData();
    } catch (IOException | DateTimeParseException e) {                              // Suppression d'SQLException
        System.err.println(e);
        throw e;
    }
}
```
On peut laisser la méthode `rethrowing()` comme elle est.
> Ces changements sont la raison pour laquelle la plupart des gens préfèrent utiliser des exceptions *unchecked*.

## Travailler avec les assertions
une *assertion* est une expression booléenne que vous placez à un moment dans votre code ou vous vous attendez à ce que quelque chose soit vrai.

une déclaration `assert` contient sa déclaration et un message optionnel.
### La déclaration `assert`
La syntaxe pour une déclaration `assert` à deux formes :
```java
assert boolean_expression;
assert boolean_expression: error_message;
```
l'expression booléenne doit renvoyer `true` ou `false`. Optionnellement, elle peut être entre parenthèses.

Une assertion lève une `AssertionError` lorsqu'elle est `false`.

Le message d'erreur optionnel est le message utilisé par l'`AssertionError`.

Les trois issues possibles pour les assertions sont les suivantes :
- Si les assertions sont désactivées, Java ignore les assertions,
- Si les assertions sont activées et que l'expression booléenne est `true`, notre assertion est validée et rien ne se passe, le programme continue à tourner normalement,
- Si les assertions sont activées et que l'expression booléenne est `false`, notre assertion est invalide et une `java.lang.AssertionError` est levée.

En présumant que les assertions sont activées, les assertions sont un meilleur / plus cours moyen d'écrire :
```java
if(!boolean_expression) throw new AssertionError();
```
Supposons que nous lancions le prochain exemple avec la commande `java -ea Assertions` :
```java
public class Assertions {
    public static void main(String[] args) {
        int numGuests = -5;
        assert numGuests > 0;
        System.out.println(numGuests);
    }
}
```
Java lève alors une `AssertionError`, le `System.out.println` n'est jamais atteint puisqu'une erreur a été levée.
```
Exception in thread "main" java.lang.AssertionError
    at asserts.Assertions.main(Assertions.java:7)
```
### Activer les assertions
Par défaut, les déclarations d'assertions sont ignorés par la JVM à l'exécution.

Pour les activer, on utilise le *flag* `-enableassertions` :
```
java -enableassertions Rectangle
```
ou le raccourcis `-ea` :
```
java -ea Rectangle
```
Utiliser le flag`-enableassertions` ou `-ea` sans argument active les assertions dans toutes les classes sauf les classes « system ».

Les classes « system » sont les classes qui font partie du *java runtime* (les classes qui viennent avec Java).

Vous pouvez aussi activer les assertions pour une classe spécifique d'un package :
```
java -ea:com.wiley.demos... my.programs.Main
```
Les trois points veulent dire « n'importe quelle classe dans un package spécifique ou sous-package »

Vous pouvez aussi activer les assertions pour une classe spécifique :
```
java -ea:com.wiley.demos.TestColors my.programs.Main
```

Vous pouvez aussi désactiver les assertions pour une classe ou un package qui étaient précédemment activées avec `-disableassertions` ou `-da` :
```
java -ea:com.wiley.demos... -da:com.wiley.demos.TestColors my.programs.Main
```
> Souvenez-vous que les assertions sont ignorées si elles ne sont pas activées.

### Utiliser les assertions
### Types d'assertions

|Nom|Description|
|---|---|
|*Internal invariants*|Vous vous assurez que la valeur est dans les bornes d'une certaine contrainte.|
|*Class invariants*|Vous vous assurez de la validité de l'état d'un objet. Ce sont généralement des méthodes privées qui renvoient un booléen.|
|*Control Flow Invariants*|Vous vous assurez qu'une ligne de code inaccessible n'est jamais atteinte.|
|*Preconditions*|Vous vous assurez que certaines conditions sont réunies avant d'appeler une méthode|
|*Post conditions*|Vous vous assurez que certaines conditions sont réunies après avoir appelé une méthode|

#### Exemple de *control flow invariant*
```java
public enum Seasons {
    SPRING, SUMMER, FALL
}
```
```java
public class TestSeasons {
    public static void test(Season s) {
        switch(s) {
            case SPRING:
            case FALL:
                System.out.println("shorter hours");
                break;
            case SUMMER:
                System.out.println("longer hours");
                break;
            default:
                assert false: "Invalid Season";
        }
    }
}
```
C'est un exemple typique d'utilisation des assertions.

Comme WINTER n'est pas dans l'`enum`, le cas par défaut n'est pas censé être atteint, Si on rajoute WINTER dans l'`enum` l'assertion nous préviendra.
```java
public enum Seasons {
    SPRING, SUMMER, FALL, WINTER
}
```
```java
public static void main(String[] args) {
    test(Seasons.WINTER);
}
```
```
Exception in thread "main" java.lang.AssertionError: Invalid season
    at TestSeason.main(Test.java:12)
    at TestSeason.main(Test.java:18)
```
Vous devriez placer des déclarations d'assertion à chaque point de votre code que vous ne pensez pas pouvoir être atteint.
#### Les assertions ne doivent pas changer le résultat
Puisque les assertions seront vraisemblablement désactivées dans l'environnement de production, vos assertions ne devraient contenir aucune logique métier (*business logic*) qui affecte le résultat de votre code.
```java
int x = 10;
assert ++x > 10; // Pas une bonne pratique
```
#### Exemple *class invariant*
```java
public class Rectangle {
    private int width, height;

    public Rectange(int width, int height) {
        this.witdth = width;
        this.height = height;
    }

    public int getArea() {
        assert isValid(): "Not a valid Rectangle";
        return width * height;
    }

    private boolean isValid() {
        return (width >= 0 && height >= 0);
    }

    public static void main(String[] args) {
        Rectangle one = new Rectangle(5, 12);
        Rectangle two = new Rectangle(-4, 10);

        System.out.println("Area one = " + one.getArea());
        System.out.println("Area two = " + two.getArea());
    }
}
```
La méthode `isValid` est un exemple de *class invariant*. C'est une méthode privée qui teste l'état de l'objet.
```
Area one = 60
Exception in thread "main"java.lang.AssertionError: Not a valid Rectangle"
    at Rectangle.getArea(Rectangle.java:10)
    at Rectangle.main(Rectangle.java:22)
```
#### Valider des paramètres de méthode
N'utilisez pas les assertions pour vérifier les paramètres d'une méthode. Utilisez plutôt l'`IllegalArgumentException`.
```java
public Rectangle(int width, int height) {
    if(width <  0 || height < 0 {
       throw new IllegalArgumentException();
    } 
    this.witdth = width;
    this.height = height;
}
```
Les assertions sont faites pour débugger, elles vous permettent de vérifier que quelque chose que vous pensez être vrai durant la phase de développement est vrai à l'exécution. 