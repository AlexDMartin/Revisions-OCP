# NIO.2

L'API `java.nio` version 2 ou NIO.2 interagit avec les fichiers.

NIO.2 est l'acronyme de *Non-blocking Input/Output API*.

> Lorsque l'on parle de « streams » dans ce chapitre, on parle de la nouvelle spécification de programmation fonctionnelle.

## Introduction à NIO.2

Java a présenté un remplacement pour les streams de `java.io` en Java 1.4, appelés « Non-blocking I/O » ou « NIO ».

L'API NIO a introduit les concepts de *buffers* et de *channels* à la place.

L'idée est vous chargez les données d'un *channel* de fichier vers un *buffer* temporaire, qui, contrairement aux streams de `byte`, peut être lu dans les deux
sens sans bloquer la ressource sous-jacente.

> Malheureusement, l'API NIO sortie en Java 1.4 n'a jamais été très populaire.

Java 7 a introduit l'API NIO.2.

Bien que l'API NIO soit un remplacement des streams de `java.io`, l'API NIO.2 est un remplacement de la classe `java.io.File` et de ces interactions.

Le but de l'implémentation de l'API NIO.2 est de fournir une API pour travailler avec les fichiers ayant plus de fonctionnalités et étant plus intuitive.

### Introduction à `Path`

L'interface `java.nio.file.Path` est le point d'entrée principal pour travailler avec l'API NIO.2.

Un objet `Path` représente un chemin hiérarchique sur l'espace de stockage vers un fichier ou un dossier.

De cette manière, `Path` remplace `java.io.File` et conceptuellement contiennent les mêmes propriétés.

Les deux peuvent utiliser un chemin absolu ou un chemin relatif dans le système de fichier.

Contrairement à la classe `File `, `Path` peut utiliser les *liens symboliques*. Un *lien symbolique* est un fichier spécial dans un système d'exploitation qui
sert de référence ou de pointeur vers un autre fichier ou dossier.

Les *liens symboliques sont transparents pour l'utilisateur.

L'API NIO.2 inclut un support complet pour la création, détection et navigation des liens symboliques dans le système de fichiers.

### Création d'instances avec les *Factory* et les classes utilitaires

Vous pouvez créer une instance de l'interface `Path` en utilisant les méthodes statiques disponibles dans la classe `Paths` (avec un `s` pour ne pas confondre).

> La raison pour laquelle `Path` est une interface et pas un classe est que la création d'un fichier est dépendant du système.
> L'avantage d'utiliser le *pattern factory* est que vous pouvez écrire le même code sur différentes plateformes.

NIO.2 inclut aussi des classes utilitaires comme `java.nio.file.File` dont le but principal est d'opérer sur les instances de `Path`.

Les classes utilitaires sont similaires aux classes *factory* car elles sont composées majoritairement de méthodes statiques qui opèrent sur une classe
particulière.

Elles diffèrent par le fait que les classes utilitaires sont centrées sur la manipulation qou la création de nouveaux objets à partir d'instances existantes
alors que les classes *factory* sont seulement centrées sur la création d'objets.

### Création des paths

Puisque `Path` est une interface, vous avez besoin d'un classe *factory* pour en créer une instance.

#### Utiliser la classe `Paths`

##### `Paths.get(String)`
La manière la plus simple d'obtenir un objet `Path` est d'utiliser la classe *factory* `java.nio.file.Paths`.

Vous aurez besoin d'appeler la méthode statique `Paths.get(String)` comme ceci :

```java
Path path1=Paths.get("pandas/cuddly.png");
Path path2=Paths.get("C:\\zooinfo\\November\\employees.txt");
Path path3=Paths.get("/home/zoodirector");
```

##### Comment déterminer si un path est relatif ou absolu
- Si le path commence par un `/`, il est absolu (eg. `/bird/parrot`)
- Si le path commence par la lettre de disque, il est absolu (eg. `C:\bird\emu`)
- Sinon il est relatif (eg. `..\eagle`)

##### `Paths.get(String...)`
Vous pouvez aussi créer un `Path` en utilisant la classe `Paths` avec un vararg de type `String` dans lequel le séparateur est directement inséré.
```java
Path path1 = Paths.get("pandas", "cuddly.png");
Path path2 = Paths.get("c:", "zooinfo", "November", "employee.txt);
Path path3 = Paths.get("/", "home", "zoodirector");
```

> Faites attention à la différence entre `Path` et `Paths` à l'examen
> ```java
> Paths path1 = Paths.get("/alligator/swim.txt")    // ❌ Ne compile pas
> Path path2 = Path.get("/crocodile/food.png")      // ❌ Ne compile pas
> ```

##### `Paths.get(URI)`
Une autre façon de créer un Path en utilisant la classe `Paths` est avec une valeur *URI*.

Une *Uniform Ressource Identifier (URI)* est une chaîne de caractère qui identifie une ressource.

Elles commencent par une *valeur schéma* qui indique le type de ressource, suivi du chemin.

Exemple de *valeur schéma* : `file://`, `http://`, `https://`, `ftp://`.

```
Path path1 = Paths.get(new URI("file://pandas/cuddly.png")); // ❌ Lève une exception à l'exécution
Path path2 = Paths.get(new URI("file:///c:/zoo-info/November/employees.txt"));
Path path3 = Paths.get(new URI("file:///home/zoodirectory"));
```

Une URI doit référencer un chemin absolu à l'exécution.

La classe `URI` possède une méthode `isAbsolute()` mais n'a rien à voir avec l'emplacement du fichier, elle renvoie juste si l'URI possède une valeur schéma.

##### Fichiers non locaux
Voici deux méthodes pour accéder à des fichiers non locaux.

Pour l'examen vous n'avez pas à connaître la syntaxe de ces schémas mais vous devez savoir qu'ils existent.

```java
Path path4 = Paths.get(new URI("http://www.wiley.com"));
Path path5 = Paths.get(new URI("ftp://username:password@ftp.the-ftp-server.com"));
```

Notez que le constructor `URI(String)` lève une exception *checked* `URISyntaxException` qui devrait être catchée dans nos exemples précédents

Finalement, L'interface `Path` contient une méthode `toUri()` pour convertir une instance de `Path` en `URI`.
```java
Path path4 = Paths.get(new URI("http://www.wiley.com"));
URI uri4 = path4.toUri();
```

#### Accéder à l'objet `FileSystem` sous-jacent
La méthode `Paths.get()` vue précédemment est en fait un raccourci vers la méthode `getPath()` de la classe `java.nio.file.FileSystem`.

La classe `FileSystem` possède un constructeur *protected*, donc nous utilisons plusieurs classes *factory* `FileSystems` pour obtenir une instance de `FileSystem`.

```java
Path path1 = FileSystems.getDefault().getPath("pandas/cuddly.png");
Path path2 = FileSystems.getDefault().getPath("c:", "zooinfo", "November", "employee.txt);
Path path3 = FileSystems.getDefault().getPath("/home/zoodirector");
```

Bien que la plupart des objets `Path` que nous souhaitons accéder sont dans le système de fichier local, cette méthode nous permet de connecter à un système de fichier distant (*remote*).

```java
FileSystem fileSystem = FileSystems.getFileSystem(new URI("http://www.selikoff.net");
Path path = fileSystem.getPath("duck.txt");
```

#### Travailler avec des instances de file *legacy*
Lorsque `Path` a été ajouté en Java 7, la classe *legacy* `java.io.File` a été mise à jour avec une nouvelle méthode `toPath()` qui opère sur une variable d'instance `File`;
```java
File file = new File("pandas/cuddly.png");
Path path = File.toPath();
```

Pour la rétrocompatibilité, l'interface `Path` contient une méthode `toFile` pour retourner une instance de `File`.
```java
Path path = Paths.get("cuddly.png");
File file = path.toFile();
```

## Interagir avec les *paths* et les fichiers
### Objet `Path` vs fichiers réels
Un objet `Path` n ést pas un fichier mais une représentation de son emplacement dans le système de fichier.

La plupart des opérations disponibles dans `Path` et `Paths` peuvent être accomplis sans savoir si le fichier que l'objet `Path` référence existe.

Comme vous le verrez, certaines opérations requièrent que ce fichier existe.

### Fournir des arguments optionnels
#### Arguments communs dans NIO.2
|Valeur de l'*enum*|Utilisation|Description|
|---|---|---|
|`NOFOLLOW_LINKS`|Teste si le fichier existe, lit les données du fichier, copie le fichier, déplace le fichier|Si fourni, les liens symboliques rencontrés ne seront pas traversés. Pratique lorsque l'on travaille sur les liens symboliques plutôt que sur leurs cibles.|
|`FOLLOW_LINKS`|Traverse l'arborescence d'un dossier|Si fourni, les lien symboliques rencontrés seront traversés.|
|`COPY_ATTRIBUTES`|Copie le fichier|Si fourni, toutes les métadonnées du fichier seront copiées avec lui.|
|`REPLACE_EXISTING`|Copie le fichier, déplace le fichier|Si fourni et que la fichier cible existe, il sera remplacé. Dans l'autre cas, une exception sera levée si le fichier existe déjà.|
|`ATOMIC_MOVE`|Déplace le fichier|L'opération sera menée de manière atomique dans le système de fichier, assurant que les processus utilisant le fichier ne voit qu'un enregistrement complet. Les méthodes l'utilisant peuvent lever une exception si le système de fichier ne supporte pas cette fonctionnalité.|

Par simplicité, le nom des enum a été omis car elles ne sont pas à l'examen.

Les méthodes de copie prennent des valeurs de l'interface `CopyOptions` dont `StandardCopyOption` est une enum implémentant cette interface (eg. `StandardCopyOption.COPY_ATTRIBUTES`).

Une *opération atomique* est une opération qui est opérée comme une unique unité d'exécution indivisible qui apparaît pour le reste du système comme instantanée.

De plus, un *mouvement atomique* est un mouvement dont tous les processus suivant ce fichier ne voient jamais des données incomplètes ou écrites partiellement.

Si le système de fichier ne supporte pas les mouvements atomiques, une `AtomicMoveNotSupportedException` sera levée.

> Les auteurs de Java ont utilisé des varargs en prévision pour le futur.

### Utiliser les objets `Path`
Vous pouvez chaîner les méthodes sur l'objet `Path`.

```java
Paths.get("/zoo/../home").getParent().normalize().toAbsolutePath();
```

> Généralement le nom de la méthode décrit la fonctionnalité

#### Consulter le `Path` avec `toString()`, `getNameCount()` et `getName()`
La méthode `toString()` est la seule méthode de `Path` qui retourne une `String` et retourne une représentation du path entier.

Les méthodes `getNameCount()` et `getName(int)` sont utilisées en conjonction pour respectivement récupérer le nombre d'éléments dans le `Path` et une référence à chaque élément.

La méthode `getName(int)` retourne le composant du `Path` comme un nouvel objet `Path` plutôt qu'une `String`.

```java
Path path = Paths.get("/land/hippo/harry.happy");
System.out.println("The Path Name is: " + path);

for(int i = 0; i < path.getNameCount(); i++) {
    System.out.println("    Element " + i + " is: " + path.getName(i));
}
```

Appeler `print` ou `println` sur un objet appelle automatiquement la méthode `toString()`
```
The Path Name is: /land/hippo/harry.happy
    Element 0: land
    Element 1: hippo
    Element 2: harry.happy
```

Remarquez que l'élément racine `/` n'est pas inclus dans la liste de noms. Si l'objet `Path` représente l'élément root, alors le nombre de noms dans l'objet `Path` retourné par `getNameCount()` sera 0.

Si on remplaçait le path précédent par le path relatif `land/hippo/harry.happy`, le résultat serait :
```
The Path Name is: land/hippo/harry.happy
    Element 0: land
    Element 1: hippo
    Element 2: harry.happy
```

Pour l'examen, vous devez savoir que `getName(int)` par de zéro, avec la racine du système de fichier ignoré dans les composants du path.

#### Accéder aux composants du `Path` avec `getFileName()`, `getParent()` et `getRoot()`

##### `getFileName()`
La méthode `getFileName()` retourne une instance de `Path` représentant le nom du fichier, qui est l'élément le plus éloigné de root.

Comme la plupart des méthodes de l'interface `Path`, `getFileName()` retourne une nouvelle instance de `Path` plutôt qu'une `String`.

##### `getParent()`
La méthode `getParent()` retourne une instance de `Path` représentant le path parent ou null s'il n'en a pas. Si l'instance de l'objet `Path` est relatif, cette méthode s'arretera à l'élément le plus élevé de l'objet `Path`.

En d'autres mots elle ne traversera pas en dehors de l'espace de travail ou de la racine du système.

##### `getRoot()`
La méthode `getRoot()` retourne l'élément racine du `Path`ou null si l'objet `Path` est relatif.

##### Exemple
```java
import java.nio.file.*;

public class PathFileTest {
    public static void printPathInformation(Path path) {
        System.out.println("Filename is: " + path.getFileName());
        System.out.println("Root is: " + path.getRoot());
        
        Path currentParent = path;
        while((currentParent = currentParent.getParent()) != null) {
            System.out.println("    Current parent is: " + currentParent);
        }
    }
    
    public static void main(String[] args) {
        printPathInformation(Paths.get("/zoo/armadillo/shells.txt"));
        System.out.println();
        printPathInformation(Paths.get("armadillo/shells.txt"));
    }
}
```
```
Filename is: shells.txt
Root is: /
    Current parent is: /zoo/armadillo
    Current parent is: /zoo
    Current parent is: /
    
Filename is: shells.txt
Root is: null
    Current parent is: armadillo
```

#### Vérifier le type de `Path` avec `isAbsolute()` et `toAbsolutePath()`
#####  `isAbsolute()`
La méthode `isAbsolute()` retourne `true`si le chemin vers lequel l'objet pointe est absolu et `false` s'il est relatif

##### `toAbsolutePath()`
La méthode `toAbsolutePath()` converti un objet `Path` relatif en objet `Path` absolu en faisant la jonction avec le répertoire de travail courant.

Si le `Path` est déjà absolu, alors la méthode renvoie une copie équivalente de ce `Path`.

##### Exemple
```java
Path path1 = Paths.get("C:\\birds\\egret.txt");
System.out.println("Path1 is Absolute ?" + path1.isAbsolute());
System.out.println("Absolute Path1: " + path1.toAbsolutePath());

Path path2 = Paths.get("birds/condor.txt");
System.out.println("Path2 is Absolute ?" + path2.isAbsolute());
System.out.println("Absolute Path2: " + path2.toAbsolutePath());
```
```
Path1 is Absolute: true
Absolute Path1: C:\birds\egret.txt

Path2 is Absolute: false
Absolute Path2: /home/birds/condor
```

##### Dépendance au système
```java
System.out.println(Paths.get("/stripes/zebra.exe").isAbsolute()); // true sur Linux et Mac, false sur Windows
System.out.println(Paths.get("c:/goats/Food.java").isAbsolute()); // true sur Windows, false sur Linux et Mac
```

#### Créer un nouveau `Path` avec `subpath()`
La méthode `subpath(int, int)` retourne un sous-chemin relatif à l'objet `Path`, référencé par un index de départ **inclus** et un index de fin **exclu**.

```java
Path path = Paths.get("/mammal/carnivore/raccoon.image");
System.out.println("Path is: "+ path);

System.out.println("Subpath from 0 to 3 is: " + path.subpath(0, 3));
System.out.println("Subpath from 1 to 3 is: " + path.subpath(1, 3));
System.out.println("Subpath from 1 to 2 is: " + path.subpath(1, 2));
```

Vous pouvez remarquer que les méthodes `subpath()` et `getName()` sont similaires puisqu'elles retournent un objet `Path` qui représente un composant d'un `Path` existant.

La différence est que la méthode `subpath()` inclut plusieurs composants path, alors que la méthode `getName()` n'en inclut qu'un.

```
Path is: mammal/carnivore/raccoon.image
Subpath from 0 to 3 is: mammal/carnivore/raccoon.image
Subpath from 1 to 3 is: carnivore/raccoon.image
Subpath from 1 to 2 is: carnivore
```

Cet exemple montre que la méthode `subpath(int, in)` n'inclut pas la racine du fichier.

```java
System.out.println("Subpath from 0 to 4 is: " + path.subpath(0, 4));    // ❌ Lève une exception à l'exécution - index maximum dépassé
System.out.println("Subpath from 1 to 1 is: " + path.subpath(1, 1));    // ❌ Lève une exception à l'exécution - début et fin pareil (path vide)
```

#### Utiliser les symboles de `Path`
Beaucoup de systèmes d'exploitation support des chemins qui contiennent des informations de chemins relatifs sous la forme de symboles.

|Symbole|Description|
|---|---|
|`.`|Une référence au répertoire courant.|
|`..`|Une référence au parent du répertoire courant.|

#### *Dériver* un `Path` avec `relativize()`
La méthode `relativize(Path)` construit un objet `Path` à partir d'un autre objet `Path`.

```java
Path path1 = Paths.get("fish.txt");
Path path2 = Paths.get("birds.txt");

System.out.println(path1.relativize(path2));
System.out.println(path2.relativize(path1));
```
```
../birds.txt
../fish.txt
```

Si les deux chemins sont relatifs alors la méthode `relativize()` compare les chemins comme s'ils étaient dans le même répertoire de travail.

Si les deux chemins sont absolus, alors la méthode `relativize()` crée un chemin relatif de l'un à l'autre et ne prend pas en compte le répertoire de travail.

```java
Path path3 = Paths.get("E:\\habitat");
Path path4 = Paths.get("E:\\sanctuary\\raven");

System.out.println(path3.relativize(path4));
System.out.println(path4.relativize(path3));
```
```
..\sanctuary\raven
..\..\habitat
```

##### Problème de compatibilité avec `relativize()`
La méthode `relativize(Path)` requiert que les `Path` soient tous les deux absolus ou tous les deux relatifs, sinon une `IllegalArgumentException` est levée à l'exécution.

```java
Path path1 = Paths.get("/primate/chimpanzee");
Path path2 = Paths.get("bananas.txt");

path1.relativize(path2); // ❌ Lève une exception à l'exécution
```

Sur les systèmes Windows, les deux `Path` doivent avoir la même racine ou même lettre de disque.
```java
Path path3 = Paths.get("c:\\primate\chimpanzee");
Path path4 = Paths.get("d:\\storage\\bananas.txt");

path3.relativize(path4); // ❌ Lève une exception à l'exécution
```

#### Joindre des objets `Path` avec resolve()
La méthode `resolve(Path)` crée un nouveau `Path` en joignant un `Path` existant au `Path` courant.

L'objet sur lequel `resolve()` est appelé devient la base de l'objet `Path` créé, avec l'argument d'entrée ajouté.

```java
final Path path1 = Paths.get("/cats/../panther");
final Path path2 = Paths.get("food");

System.out.println(path1.resolve(path2));
```
```
/cats/../panther/food
```

Pour l'examen, vous devez savoir que, comme la méthode `relativize()`, la méthode `resolve()` ne nettoie pas les symboles du path. Pour cela vous devrez utiliser `normalize()`.

##### Exemple avec un path absolu
```java
final Path path1 = Paths.get("/turkey/food");
final Path path2 = Paths.get("/tiger/cage");

System.out.println(path1.resolve(path2));
```
```
/tiger/cage
```
Dans le cas d'un path absolu, `path1` est ignoré et une copie de `path2` est retournée.

#### Nettoyer un `Path` avec `normalize()`
Java fournit une méthode `normalize(Path)` qui élimine les redondances dans le path

```java
Path path3 = Paths.get("E:\\data");
Path path4 = Paths.get("E:\\user\\home");

Path relativePath = path3.relativize(path4);
System.out.println(path3.resolve(relativePath));
```
```
E:\data\..\user\home
```
Maintenant avec `normalize()` :
```java
System.out.println(path3.resolve(relativePath).normalize());
```

```
E:\user\home
```

#### Vérifier l'existence d'un fichier avec `toRealPath()`
La méthode `toRealPath(Path)` prend un objet `Path` qui peut pointer vers un fichier (ou pas) du système de fichier, et retourne une référence vers le vrai chemin du fichier dans système de fichier.

Elle est similaire à la méthode `toAbsolutePath()` puisqu'elle convertit aussi les chemins relatifs en chemins absolus, sauf qu'elle vérifie aussi que le fichier référencé par le chemin existe.

Si le fichier n'est pas localisé, un `IOException` est levée.

C'est la seule méthode de `Path` qui supporte l'option `NOFOLLOW_LINKS`.

La méthode `toRealPath` supprime aussi les éléments redondants du chemin. En d'autres mots, elle appelle implicitement la méthode `normalize()` sur le chemin absolu retourné.

##### Exemples
Considérons le lien symbolique suivant 
```
/zebra/food.source -> /horse/food.txt
```
En considérant que notre repertoire de travail est `/horse/schedule`

```java
try {
    System.out.println(Paths.get("/zebra/food.source").toRealPath());
    System.out.println(Paths.get(".././food.txt").toRealPath());
} catch (IOException e) {
    // Gérer la IOException
}
```

```
/horse/food.txt
/horse/food.txt
```

Finalement nous pouvons utiliser `toRealPath()` pour récupérer le chemin du répertoire courant.
```java
System.out.println(Paths.get(".").toRealPath());
```

### Interagir avec les `Files`
La plupart des opérations disponibles dans `java.io.File` sont disponibles avec `java.nio.files.Path` par le biais de la classe utilitaire `java.nio.file.Files`

> Encore une fois attention à ne pas confondre `File` et `Files` (de même que `Collection` et `Collections` / `Path` et `Paths`).

La classe `Files` contient plusieurs méthodes statiques permettant d'interagir avec les fichiers, la plupart prenant un ou deux arguments `Path`.

Certaines de ces méthodes sont capables de lever une `IOException` à l'exécution, la plupart du temps lorsque le fichier référencé n'existe pas.

#### Tester un Path avec `exists()`
La méthode `Files.exists(Path)` prend un objet `Path` et retourne `true` si, et seulement si, il référence un objet qui existe dans le système de fichier.

```java
// Exemple avec un fichier
Files.exists(Paths.get("/ostrich/feathers.png"));

// Exemple avec un dossier
Files.exists(Paths.get("/ostrich"));
```

#### Tester l'unicité avec `isSameFile()`
La méthode `Files.isSameFile(Path, Path)` est utile pour déterminer si deux objets `Paths` se réfèrent au même fichier dans le système de fichier.

Elle prend deux objets `Path` et suit les liens symboliques.

Malgré son nom, elle peut aussi être utilisée pour des dossiers.

La méthode `isSameFile` vérifie d'abord si les deux `Path` sont équivalents en termes d'`equal()`, si oui elle renvoie directement `true` sans vérifier que le fichier existe.

Si l'`equal()` renvoie `false`, alors elle localise les fichiers avant de déterminer si ce sont les mêmes, et lève une `IOException` si l'un des dux fichier n'existe pas.

> La méthode `isSameFile()` ne compare pas le contenu des fichiers.
> Si les fichiers ont des attributs et un contenu identique mais qu'ils sont à différents endroits `isSameFile()` renverra `false`

Considérons que les deux fichiers existent dans le système de fichier et que `cobra` est un lien symbolique vers `snake`.

```java
try {
    System.out.println(Files.isSameFile(Paths.get("/user/home/cobra"), Paths.get("/user/home/snake")));         // ✔ true
    System.out.println(Files.isSameFile(Paths.get("/user/tree/../monkey"), Paths.get("/user/monkey")));         // ✔ true
    System.out.println(Files.isSameFile(Paths.get("/leaves/./giraffe.exe"), Paths.get("/leaves/giraffe.exe"))); // ✔ true
    System.out.println(Files.isSameFile(Paths.get("/flamingo/tail.data"), Paths.get("/cardinal/tail.data")));   // ❌ false
} catch (IOException e) {
    // Gérer l'IOException
}
```

#### Créer des répertoires avec `createDirectory()` et `createDirectories()`
Dans l'API legacy `java.io` nous utilisions `mkdir()` et `mkdirs()` sur des objets `Files`.

Dans l'API NIO.2 nous utilisons `Files.createDirectory(Path)` pour créer un dossier.

Il existe aussi la version plurielle `Files.createDirectories()` qui, comme `mkdirs()`, crée le dossier cible ainsi que tous les dossiers parents non-existants du path.

Les méthodes de création de dossier peuvent lever un `IOException`, par exemple lorsque le dossier ne peut être créé ou lorsqu'il existe déjà. De plus `createDirectory()` lèvera cette exception si le dossier parent n'existe pas.

Ces deux méthodes acceptent une liste optionnelle de valeur `FileAttribute<?>` à passer sur nos/notre dossier/s à la création.

```java
try {
    File.createDirectory(Paths.get("/bison/field"));
    File.createDirectories(Paths.get("/bison/field/pasture/green"));
} catch (IOException e) {
    // Gérer l'IOException
}
```

#### Dupliquer des contenus de fichiers avec `copy()`
Contrairement à la classe legacy `java.io.File`, la classe `Files` de NIO.2 fournit un ensemble de surcharges de la méthode `copy()`.

La principale à connaitre pour l'examen est `Files.copy(Path, Path)` qui copie un fichier d'un dossier vers un autre.

La méthode `copy()` lève une exception *checked* `IOException`, par exemple lorsque le fichier n'existe pas ou ne peut pas être lu.

Les copies de dossier sont superficielles, ce qui veut dire que les fichiers et les sous-dossiers ne sont pas copiés.

Pour copier le contenu d'un dossier, vous devriez créer une fonction qui traverse le dossier et copie chaque fichier et sous-dossier individuellement.

```java
try {
    Files.copy(Paths.get("/panda"), Paths.get("/pandas-save"));
    Files.copy(Paths.get("/panda/bamboo.txt"), Paths.get("/pandas-save/bamboo.txt"));
} catch(IOException e) {
    // Gérer l'IOException
}
```
Par défaut, copier des fichiers et des dossiers traversera les liens symboliques mais n'écrasera pas un fichier ou un dossier pré-existant, et ne copiera pas les attributs des fichiers.

Ce comportement peut-être changé en fournissant respectivement les options additionnelles, `NOFOLLOW_LINKS`, `REPLACE_EXISTING` et `COPY_ATTRIBUTES`.

#### Copier des fichiers avec `java.io` et NIO.2
L'API Files NIO.2 contient deux méthodes `copy()` surchargées pour copier des fichiers en utilisant les streams `java.io`.

La première prend une source `java.io.InputStream` avec cible `Path`. Elle lit le contenu du stream et l'écrit en sortie dans un fichier représenté par l'objet `Path`.

La seconde prend une source `Path` avec cible `java.io.InputStream`. Elle lit le contenu du fichier et l'écrit en sortie dans le stream.

```java
try(InputStream is = new FileInputStream("source-data.txt"); OutputStream out = new FileOutputStream("output-data.txt")){
    
    // Copier les données du stream vers un fichier
    Files.copy(is, Paths.get("c:\\mammals\\wolf.txt"));
    
    // Copier les données du fichier vers le stream
    Files.copy(Paths.get("c:\\fish\\clown.xsl", out));
} catch (IOException e) {
    // Gérer l'IOException
}
```

La première méthode `copy(InputStream, Path)` prend des options vararg optionnel puisqu'il écrit vers un fichier représenté par un `Path`.

À l'inverse, la seconde méthode `copy(Path, InputStream)` n'accepte pas d'options puisqu'elle écrit vers un stream.

#### Changer l'emplacement d'un fichier avec `move()`
La méthode `Files.move(Path, Path)` déplace ou renomme un fichier ou un dossier dans le système de fichier.

Comme la méthode `copy()`, la méthode `move()` lève une exception *checked* `IOException`dans le cas ou le fichier ou le dossier n'a pas pu être trouvé ou déplacé.

```java
try {
    Files.move(Paths.get("c:\\zoo"), Paths.get("c:\\zoo-new"));
    Files.move(Paths.get("c:\\user\\addresses.txt"), Paths.get("c:\\zoo-new\\addresses.txt"));
} catch (IOException e) {
    // Gérer l'IOException
}
```

PAr défaut, la méthode `move()` suivra les liens symboliques, lèvera une exception si le fichier existe déjà et ne fera pas de mouvement atomique.

Ce comportement peut-être changé en fournissant respectivement les options additionnelles, `NOFOLLOW_LINKS`, `REPLACE_EXISTING` et `ATOMIC_MOVE`.

Si le système de fichier ne supporte pas les mouvements atomiques, une `AtomicMoveNotSupportedException` sera levée à l'exécution.

> La méthode `Files.move()` peut être appliqué aux dossiers non vides seulement s'ils sont sur le même disque.
> Bien que déplacer un dossier vide entre disque est supporté, déplacer un dossier non vide entre disque lèvera une `DirectoryNotEmptyException`.

#### Supprimer un fichier avec `delete()` et `deleteIfExists()`
##### `delete()`
La méthode `Files.delete(Path)` supprime un fichier ou un dossier vide dans le système de fichier.

La méthode `delete()` lève une exception *checked* `IOException` sous plusieurs circonstances, par exemple, si le chemin représente un dossier non-vide, une `DirectoryNotEmptyException` sera levée à l'exécution.

Si la cible du path est un lien symbolique, le lien symbolique sera supprimé mais pas la cible du lien.

##### `deleteIfExists()`
La méthode `deleteIfExists(Path)` est presque identique à la méthode `delete(Path)`, sauf qu'elle ne lèvera pas d'exception si le fichier ou le dossier n'existe pas, mais elle retournera une valeur booléenne `false`.

Elle lèvera quand même une exception si le fichier ou le dossier existe mais échoue, par exemple, si le dossier n'est pas vide.

##### Exemple
```java
try {
    Files.delete(Paths.get("/vulture/feathers.txt));
    Files.deleteIfExists(Paths.get("/pigeon));
} catch (IOException e) {
    // Gérer l'IOException
}
```

#### Lire et écrire de données du fichier avec `newBufferedReader()` et `newBufferedWriter()`
L'API NIO.2 inclut des méthodes pour lire et écrire le contenu des fichiers en utilisant les streams de `java.io`. 

##### `newBufferedReader()`
La méthode `Files.newBufferedReader(Path, Charset)` lit le fichier spécifié en utilisant un objet `java.io.BufferedReader`.

Elle à besoin d'un `Charset` pour déterminer le *character encoding* à utiliser pour lire le fichier.

```java
Path path = Paths.get("/animals/gopher.txt");
try(BufferedReader reader = Files.newBufferedReader(path, Charset.forName("US-ASCII"))) {
    // Lecture à partir du stream
    String currentLine = null;
    
    while((currentline = reader.readLine()) != null)
        System.out.println(currentLine); 
} catch (IOException e) {
    // Gérer l'IOException
}
```

Cet exemple lit le contenu du stream, comme vous le verrez plus tard il y a une façon bien plus simple de faire cela avec les streams de la programmation fonctionnelle.

##### `newBufferedWriter()`
La méthode `Files.newBufferedWriter(Path, Charset)` écrit vers le fichier spécifié en utilisant un `BufferedWriter`

```java
Path path = Paths.get("/animals/gorilla.txt");
List<String> data = new ArrayList();
try(BufferedWriter writer = Files.newBufferedWriter(path, Charset.forName("UTF-16"))) {
    writer.write("Hello World");
} catch (IOException e) {
    // Gérer l'IOException
}
```

Le `newBufferedWriter()` peut aussi prendre un vararg avec des valeurs d'`enum` pour ajouter au fichier plutôt que d'écraser.

#### Lire des fichiers avec `readAllLines()`
La méthode `Files.readAllLines()` lis toutes les lignes d'un fichier texte et retourne le résultat dans une list ordonnée de `String`.

L'API NIO.2 possède aussi une version surchargée qui accepte un `Charset`.

```java
Path path = Paths.get("/fish/sharks.log");
try {
    final List<String> lines = Files.readAllLines(path);
    for(String line: lines) {
        System.out.println(line);
    }  
} catch (IOException e) {
    // Gérer l'IOException
}
```

Cette méthode lève une IOException si le fichier ne peut pas être lu.

Soyez conscients  qu'avec readAllLines() la totalité du fichier est lu et est stocké, pour un fichier suffisament gros une `OutOfMemoryError` peut être levée.