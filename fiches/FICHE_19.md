## Comprendre les attributs des fichiers

La classe `Files` permet aussi d'accéder aux *métadonnées* des fichiers et des dossiers, aussi appelées *attributs de fichier*.

Une *métadonnée* est une donnée qui décrit une autre donnée. Dans ce contexte, les *métadonnées* des fichiers sont des données à propos du fichier ou du
dossier dans le système de fichier mais ne font pas partie du contenu de ce fichier.

> Certaines opérations sur les métadonnées sont dépendantes du système.

### Découverte des attributs de fichiers basiques

#### Lire les attributs communs avec `isDirectory()`, `isRegularFile()` et `isSymbolicLink()`

Java définit un *regular file* comme fichier qui possède du contenu, en opposition aux liens symboliques, dossiers, ressource, ou tout autre fichier *
non-regular* qui peut être présent sur le système d'exploitation.

Si le lien symbolique pointe vers un vrai fichier ou dossier, Java vérifiera la cible du lien symbolique. De ce fait il est possible què `isRegularFile()`
renvoie `true` sur un lien symbolique si la cible pointe vers un fichier régulier.

##### Exemple
Supposons que `/canine/coyote` et `/canine/types.txt` existent. De plus, supposons que `/coyotes` est un lien symbolique vers autre part dans le système de fichier

```java
File.isDirectory(Paths.get("/canine/coyote/fur.jpg"));
File.isRegularFile(Paths.get("/canine/types.txt"));
File.isSymbolicLink(Paths.get("/canine/coyote"));
```

|   |`isDirectory()`|`isRegularFile()`|`isSymbolicLink()`|
|---|---|---|---|
|`/canine/coyote`|`true`|`false`|`false`|
|`/canine/types.txt`|`false`|`true`|`false`|
|`/coyotes`|`true` si la cible est un dossier|`true` si la cible est un fichier *regular*|`true`|

##### Gestion des exceptions
Les méthodes `isDirectory()`, `isRegularFile()` et `isSymbolicLink()` **ne lèvent pas** d'exception si le path n'existe pas.

Ce code est donc redondant :
```java
if(Files.exists(path) && Files.isDirectory(path)){
```
Et peut être remplacé par :
```java
if(Files.isDirectory(path)){
```

#### Vérifier la visibilité du fichier avec `isHidden()`
La méthode `Files.isHidden(Path)` détermine si le fichier ou le dossier est caché par le système de fichier.

Elle lève une `IOException` si une erreur se produit lors de la lecture des informations du fichier.

```java
try {
    System.out.println(Files.isHidden(Paths.get("/walrus.txt")));
} catch (IOException e) {
    // Gérer l'IOException
}
```

#### Tester l'accessibilité du fichier avec `isReadable()` et `isExecutable()`
Les méthodes `Files.isReadable()` et `Files.isExecutable()` permettent de lire l'accessibilité du fichier.

Comme `isDirectory()`, `isRegularFile()` et `isSymbolicLink()`, ces deux méthodes **ne lèvent pas** d'exception mais retournent `false` si le fichier n'existe pas.

```java
System.out.println(Files.isReadable(Paths.get("/seal/baby.png")));
System.out.println(Files.isExecutable(Paths.get("/seal/baby.png")));
```

#### Lire la taille du fichier avec `size()`
La méthode `File.size(Path)` est utilisée pour déterminer la taille du fichier en *bytes*.

La taille du fichier retournée par cette méthode est la taille conceptuelle des données, qui peut différer de la taille réelle du fichier à cause de la compression ou de l'organisation du système de fichier.

La méthode `size()` lève une `IOException` si le fichier n'existe pas ou si le processus n'arrive pas à lire les informations

```java
try {
    System.out.println(Files.size(Paths.get("/zoo/c/animals.txt")));
} catch (IOException e) {
    // Gérer l'IOException
}
```

> Appeler `Files.size()` sur un dossier est dépendant du système et indéfini.
> Si vous avez besoin de connaître la taille d'un dossier et de son contenu, vous aurez besoin de parcourir l'arborescence.

#### Gérer les modifications de fichier avec `getLastModifiedTime()` et `setLastModifiedTime()`
La plupart des systèmes d'exploitations supporte le suivi de la dernière *date/time* de modification pour chaque fichier.

Certaines applications utilisent cette information pour savoir si le fichier doit être relu.

##### `getLastModifiedTime()`
La méthode `Files.getLastModifiedTime(Path)` retourne un objet `FileTime` qui est un conteneur pour ces informations.

La classe `FileTime` possède une méthode statique `toMillis()`.

##### `setLastModifiedTime()`
La méthode `Files.setLastModifiedTime(Path, FileTime)` permet de mettre à jour la *date/time* de dernière modification.

La classe `FileTime` possède aussi une méthode statique `fromMillis()`.

Ces deux méthodes lèvent une exception *checked* `IOException` lorsque le fichier est accédé ou modifié.

> Vous n'avez pas à modifier le fichier pour changer la valeur de dernière modification.
> Une bonne pratique est de modifier la valeur de dernière modification lorsque les données du fichier changent.

```java
try {
    final Path path = Paths.get("/rabbit/food.jpg");
    
    System.out.println(Files.getLastModified(path).toMillis());
    
    Files.setLastModified(path, FileTime.fromMillis(System.currentTimeMillis()));
    
    System.out.println(Files.getLastModified(path).toMillis());
} catch (IOException e) {
    // Gérer l'IOException
}
```

#### Gérer les droits de propriété avec `getOwner()` et `setOwner()`
Plusieurs systèmes d'exploitations supportent la notion de fichiers et dossiers appartenant à des utilisateurs.

La méthode `Files.getOwner(Path)` retourne une instance de `UserPrincipal` qui représente le propriétaire du fichier dans le système de fichier.

Il existe aussi une méthode pour définir le propriétaire du fichier, appelée `Files.setOwner(Path, UserPrincipal)`.

Notez que le système d'exploitation interviendra lors de cette opération, par exemple, dans le cas où un utilisateur ne peut pas prendre la propriété d'un fichier.

Ces deux méthodes lèvent une exception *checked* `IOException` s'il y a des problèmes lors de l'accès au fichier.

##### `UserPrincipalLookupService`
Dans le but de donner la propriété à un utilisateur défini, NIO.2 fournit la classe utilitaire `UserPrincipalLookupService` pour trouver le `UserPrincipal` correspondant à un utilisateur particulier du système de fichier. 

```java
UserPrincipal owner = FileSystems.getDefault().getUserPrincipalLookupService().lookupPrincipalByName("jane");

Path path = ...

UserPrincipal owner = path.getFileSystem().getUserPrincipalLookupService().lookupPrincipalByName("jane");
```

##### Exemple
```java
try {
    // Lecture du propriétaire du fichier
    Path path = Paths.get("/chicken/feather.txt");
    System.out.println(Files.getOwner(path).getName();
    
    // Changement du propriétaire du fichier
    UserPrincipal owner = path.getFileSystem().getUserPrincipalLookupService().lookupPrincipalByName("jane");
    Files.setOwner(path, owner);
    
    // Affichage des informations mises à jour du propriétaire
    System.out.println(Files.getOwner(path).getName();
} catch (IOException e) {
    // Gérer l'IOException
}

```

#### Améliorer l'accès avec les vues
Il existe une manière plus efficiente de récupérer toutes les métadonnées des fichiers avec un seul appel.

De plus, certains attributs sont spécifiques au système de fichier et peuvent difficilement être généralisés pour tous les systèmes de fichier.

NIO.2 vous permet de construire des **vues** pour des différents systèmes de fichiers en un seul appel.

Une **vue** est un groupe d'attributs associés pour un type système de fichier particulier.

Un fichier peut supporter plusieurs vues, vous permettant de récupérer et de mettre à jour plusieurs sets d'informations sur ce fichier.

Le gain de performance peut être considérable lorsque l'on souhaite récupérer plusieurs attributs.

##### Comprendre les vues
Pour demander une vue, vous devez fournir :
- un path vers un fichier ou un dossier dont vous voulez lire les informations,
- un objet classe, qui permet à l'API NIO.2 de savoir quelles type de vue vous voulez.

L'API `Files` inclut deux méthodes pour accéder aux vues :
- `Files.readAttributes()` renvoie une vue des attributs en lecture seule.
- `File.getFileAttributesView()` retourne la vue des attributs et fournit un accès direct pour modifier les informations.

Ces deux méthodes lèvent une `IOException`, par exemple, quand le type de vue n'est pas supporté.

|Classe des Attributs|Classe de la vue|Description|
|---|---|---|
|`BasicFileAttributes`|`BasicFileAttributeView`|Attributs basiques supportés par tous les systèmes de fichier|
|`DosFileAttributes`|`DosFileAttributeView`|Attributs supportés par les systèmes *DOS/Windows-based*|
|`PosixFileAttributes`|`PosixFileAttributeView`|Attributs supportés par les systèmes POSIX, comme UNIX, Linux, Mac etc.|

Pour l'examen, vous devez connaître `BasicFileAttributes` et les méthodes de `BasicFileAttributeView`.

##### Lecture des attributs
La méthode `Files.readAttributes(Path, Class<A>)` renvoie une vue des attributs en lecture seule.

Le second paramètre utilise les génériques tel que la valeur de retour soit une instance de la classe fournie

###### `BasicFileAttributes`
Toutes les classes d'attributs étendent `BasicFileAttributes`, de ce fait elle contient des attributs communs à tous les systèmes de fichiers supportés.

```java
import java.io.IOException;
import java.nio.file.*;
import java.nio.file.attribute.BasicFileAttributes;

public class BasicFileAttributesSample {
    public static void main(String[] args) {
        Path path = Paths.get("/turtles/sea.txt");
        BasicFileAttributes data = Files.readAttributes(path, BasicFileAttributes.class);

        System.out.println("Is path a directory? " + data.isDirectory());
        System.out.println("Is path a regular file? " + data.isRegularFile());
        System.out.println("Is path a symbolic link? " + data.isSymbolicLink());
        System.out.println("Path not a file, directory, nor symbolic link? " + data.isOther());

        System.out.println("Size (in bytes): " + data.size());

        System.out.println("Creation date/time: " + data.creationTime());
        System.out.println("Last modified date/time: " + data.lastModifiedTime());
        System.out.println("Last accessed date/time: " + data.lastAccessTime());

        System.out.println("Unique file identifier (if available): " + data.fileKey());
    }
}
```

##### Modification des attributs
Bien que la méthode `Files.readAttributes()` soit pratique pour lire les informations du fichier, elle ne permet pas un accès direct pour les modifier.

L'API NIO.2 fournit la méthode `Files.getAttributeView(Path, Class<V>)` qui retourne une vue d'objets qui est utilisée pour modifier les attributs dépendant du système de fichier.

Nous pouvons utiliser `readAttributes()` directement sur l'objet « vue ».

###### `BasicFileAttributeView`
La méthode `BasicFileAttributeView` est souvent utilisé pour modifier les valeurs date/time, car les autres attributs basiques changeraient l'état du fichier (eg. fichier -> dossier, changement de taille etc.).

```java
import java.io.IOException;
import java.nio.file.*;
import java.nio.file.attribute.*;

public class BasicFileAttributeViewSample {

    public static void main(String[] args) {
        Path path = Paths.get("/turtles/sea.txt");

        BasicFileAttributesView view = Files.getFileAttributeView(path, BasicFileAttributeView.class);
        BasicFileAttributes data = view.readAttributes();
        
        FileTime lastModifiedTime = FileTime.fromMillis(data.lastModifiedTime().toMillis() + 10_000);
        
        view.setTimes(lastModifiedTime, null, null);
    }
}
```

Il n'existe qu'une méthode de modification `setTimes(FileTime lastModified, FileTime lastAccessTime, FileTime createTime)` dans la classe `BasicFileAttributeView`.

L'API NIO.2 vous permet de passer `null` pour chaque valeur date/time que vous ne souhaitez pas modifier.

## Présentation des nouvelles méthodes *stream*
Avant Java 8, les techniques utilisées pour effectuer des opérations complexes sur les fichiers avec NIO.2 étaient très verbeuses et vous obligeaient souvent à créer un classe entière pour réaliser un tâche simple.

Lorsque Java 9 est sorti, un nouvel ensemble de méthodes basé sur les streams a été ajouté à la spécification NIO.2 pour permettre de faire ces opérations complexes avec une seule ligne de code.

### Conceptualiser le parcours d'un dossier
Une tâche commune dans un système de fichier est d'itérer sur les descendants d'un path particulier, soit pour récupérer des informations sur eux, ou plus communément les filtrer.

Le *walking* ou *traversing* d'un dossier est le procédé par lequel vous commencez par un dossier parent puis itérez sur ces descendants jusqu'à ce qu'une certaine condition soit vérifiée ou qu'il n'y ait plus d'éléments sur lesquels itérer.

#### Sélectionner une stratégie de recherche
Il existe deux stratégies associées au parcours d'une arborescence de dossier :
- Le parcours en profondeur (*depth-first*)
- Le parcours en largeur (*breadth-first*)

##### Parcours en profondeur
Le parcours en profondeur traverse la structure depuis la racine vers une branche arbitraire puis revient vers la racine et aisi de suite.

La profondeur de recherche est la distance de la racine au noeud courant.

Pour des raisons de performances, certains processus ont un profondeur de recherche maximum.

##### Parcours en largeur
Le parcours en largeur commence par la racine et cherche ensuite tous les éléments d'une profondeur avant d'avancer vers une autre profondeur.

Les résultats sont ordonnés par profondeur.

Pour l'examen vous devez savoir que l'API Streams utilise le parcours en profondeur avec une profondeur maximum par défaut de `Integer.MAX_VALUE`

> Le parcours en profondeur tend à prendre moins de mémoire.
> D'un autre côté, le parcours en largeur est plus efficace lorsque l'élément que vous recherchez est proche de la racine.

### Parcourir un dossier
La méthode `Files.walk(path)` retourne un `Stream<Path>` qui traverse le dossier en profondeur de manière *lazy*.

Par *lazy*, nous entendons que le set d'éléments est créé et lu pendant que le dossier est en train d'être traversé.

> Sachez que lorsque vous créez un objet `Stream<Path>` en utilisant `Files.walk(), Le contenu du dossier n'a pas encore été traversé

```java
Path path = Paths.get("/bigcats");

try {
    Files.walk(path)
        .filter(p -> p.toString().endsWith(".java")
        .forEach(System.out::println);
} catch(IOException e) {
    // Gérer l'IOException
}
```
```
/bigcats/version1/backup/Lion.java
/bigcats/version1/Lion.java
/bigcats/version1/Tiger.java
/bigcats/Lion.java
```

Par défaut, la méthode itère jusqu'à `Integer.MAX_VALUE`.

Vous pouvez changer ce comportement en utilisant la surcharge `walk(Path, int)` qui prend la profondeur maximale comme second argument.

Une valeur de `0` indique « uniquement le path lui-même ».

> Java limite la profondeur maximum à `Integer.MAX_VALUE` car la plupart des systèmes de fichier ne supporte pas plus.

> L'objet `DirectoryStream<Path>` fournit par la méthode `Files.newDirectoryStream()` qui fonctionne de manière similaire à `walk()` n'est pas un stream car il ne descend pas de `java.util.stream.Stream`.

#### Éviter les chemins circulaires
Contrairement aux méthodes de NIO.2, la méthode walk ne traversera pas les liens symboliques par défaut.

Vous pouvez changer ce comportement en fournissant l'option `FOLLOW_LINKS` au vararg de la méthode `walk`.

Il est recommandé de mettre une limite de profondeur car il pourrait y avoir des cycles, et cela retournerait une `FileSystemLoopException`.

### Rechercher dans un dossier
Dans l'exemple précédent, nous avons appliqué `filter()` sur l'objet `Stream<Path>` pour filter les résultats. Or, NIO.2 nous propose une méthode plus directe.

La méthode `Files.find(Path, int, BiPredicate)` se comporte comme la méthode `Files.walk()`, sauf qu'elle requiert que la profondeur de recherche soit explicitement définie avec un BiPredicate pour filtrer les données.

Comme `walk()`, `find()` supporte l'option vararg `FOLLOW_LINK`.

Un `BiPredicate` est une interface qui prend deux objets génériques et renvoie une valeur booléenne sous la forme `(T, U) -> boolean`.

Dans ce cas, les deux types d'objet sont `Path` et `BasicFileAttributes`.

De cette manière NIO.2 charge automatiquement l'objet `BasicFileAttributes` pour vous, vous permettant d'écrire des expressions lambda qui ont un accès direct vers cet objet.

```java
Path path = Paths.get("/bigcats");
long dateFilter = 142007040000l;

try {
    Stream<Path> stream = Files.find(path, 10, (p, a) -> p.toString().endsWith(".java") && a.lastModifiedTime().toMillis() > dateFilter);
    
    stream.forEach(System.out::println);
} catch(IOException e) {
    // Gérer l'IOException
}
```

### Lister le contenu d'un dossier
Avec `java.io` nous avons présenté la méthode `listFiles()` qui opérait sur une instance de `java.io.File` et retourne une liste d'objets `File`.

Bien que vous puissiez utiliser la méthode `Files.walk()` avec une limite de profondeur de 1 pour lister le contenu d'un dossier, l'API NIO.2 inclut une nouvelle méthode `Files.list(Path)`, qui fait cela pour vous.

```java
try {
    Path path = Paths.get("ducks");
    Files.list(path)
        .filter(p -> !Files.isDirectory(p))
        .map(p -> p.toAbsolutePath())
        .forEach(System.out::println)
} catch(IOException e) {
    // Gérer l'IOException
}
```
```
/zoo/ducks/food.txt
/zoo/ducks/food-backup.txt
/zoo/ducks/weight.txt
```

### Affichage du contenu du fichier
Plus tôt dans ce chapitre, nous avons présenté `Files.readAllLines()` et avons commenté que l'utiliser pouvait mener à une `OutOfMemoryError` lorsqu'elle était appelée sur de gros fichiers.

Heureusement, l'API NIO.2 propose une méthode `Files.lines()` qui retourne un `Stream<String>` qui ne souffre pas du même problème.

```java
// Exemple basique
Path path = Paths.get("/fish/sharks.log");
try {
    Files.lines(path).forEach(System.out::println);
} catch(IOException e) {
    // Gérer l'IOException
}
```
```java
// Exemple plus élaboré
Path path = Paths.get("/fish/sharks.log");
try {
    System.out.println(
        Files.lines(path)
            .filter(s -> s.startsWith("WARN "))
            .map(s -> s.substring(5))
            .collect(Collectors.toList())
    );
} catch(IOException e) {
    // Gérer l'IOException
}
```
#### `Files.readAllLines()` vs `Files.lines()`
Pour l'examen, vous devez savoir que `readAllLines()` retourne une `List` et `lines()` qui retourne un `Stream`.

Cela peut être perturbant car `forEach` peut être appelé sur les deux. Ces deux exemples compilent :
```java
Files.readAllLines(Paths.get("birds.txt)).forEach(System.out::println);

Files.lines(Paths.get("birds.txt)).forEach(System.out::println);
```
La première ligne stocke tout le fichier en mémoire et applique une opération d'affichage sur le résultat.

La seconde lis les lignes de façon *lazy* et les affiche en même temps. L'avantage de cette approche est que tout le fichier n'a pas besoin d'être lu.

Pour l'examen, Vous devrez reconnaître lorsque deux types incompatibles sont mélangés :
```java
Files.readAllLines(path).filter(s - s.length() > 2).forEach(System.out::println);   // ❌ Ne compile pas - filter n'est pas une méthode de collection

Files.lines(path).filter(s - s.length() > 2).forEach(System.out::println);          // ✔ Compile
```

## Comparaison des méthodes de `File` *legacy* et NIO.2

|Méthode *legacy*|Méthode NIO.2|
|---|---|
|`file.exists()`|`Files.exists(path)`|
|`file.getName()`|`path.getFileName()`|
|`file.getAbsolutePath()`|`path.toAbsolutePath()`|
|`file.isDirectory()`|`Files.isDirectory(path)`|
|`file.isFile()`|`Files.isRegularFile(path)`|
|`file.isHidden()`|`Files.isHidden(Path)`|
|`file.length()`|`Files.size(path)`|
|`file.lastModified()`|`Files.getLastModifiedTime(path)`|
|`file.setLastModified(time)`|`Files.setLastModified(path, fileTime)`|
|`file.delete()`|`Files.delete(path)`|
|`file.renameTo(otherFile)`|`Files.move(path, otherPath)`|
|`file.mkdir()`|`Files.createDirectory(path)`|
|`file.mkdirs()`|`Files.createDirectories(path)`|
|`file.listFiles()`|`Files.list(path)`|