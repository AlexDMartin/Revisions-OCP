# IO

## Comprendre les fichiers et les dossiers

### Conceptualisation du système de fichier

Un *fichier* (*file*) est un enregistrement du système de fichier qui stocke les données de l'utilisateur et du système.

Les fichiers sont organisés dans des dossiers (*directories*).

Un dossier est un enregistrement du système de fichier qui contient des fichiers ou d'autres dossiers.

Le dossier *root* est le dossier le plus haut du système de fichier, depuis lequel héritent tous les dossier et tous les fichiers. Sur Windows, il est dénommé
avec le nom du disque eg. `C:\`, alors que sur Linux il se nomme `/`.

Dane le but d'interagir avec ds fichiers, nous devons nous connecter au système de fichier. Le système de fichier est chargé de la lecture et de l'écriture des
données dans un ordinateur.

Différents systèmes d'exploitation utilisent des systèmes de fichier différents pour gérer leurs données.

Un *chemin* (*path*) est une chaîne de caractère représentant un dossier ou un fichier dans le système de fichier. Chaque système de fichier définit ses propres
caractères séparateurs qui est utilisé entre les différentes entrées de dossier.

Dans la plupart des systèmes de fichier, la valeur à gauche du séparateur est le parent de celle de droite.

### Introduction à la classe `File`

La classe `File` est la plus utilisée de l'API `java.io`.

Elle est utilisée pour lire les informations sur des fichiers ou des dossiers existants, lister le contenu d'un dossier, et créer / supprimer des fichiers et
des dossiers.

Une instance de la classe `File` représente le *pathname* d'un unique fichier ou dossier dans le système de fichier.

La classe `File` ne peut pas lire ou écrire des données dans un fichier, même si elle peut être passée comme référence aux méthodes le permettant.

> La principale erreur des développeurs Java est d'oublier que la classe `File` peut aussi représenter des dossiers.

#### Créer un objet `File`

Un objet `File` est souvent initialisé avec une chaîne de caractère contenant soit un chemin absolu soit un chemin relatif au dossier ou au fichier du système
de fichier.

Le chemin absolu d'un dossier ou d'un fichier est le chemin complet depuis le répertoire *root*, en incluant tous les sous-dossiers contenant ce fichier ou
dossier.

Le chemin relatif d'un dossier ou d'un fichier est le chemin depuis le dossier de travail courant.

```
Exemple de chemin absolu
/home/smith/data/zoo.txt

Exemple de chemin absolu
data/zoo.txt
```

Différents systèmes d'exploitation varient dans le format de leurs chemins. Par exemple, les systèmes Unix-based utilisent `/` pour leurs chemins alors que les
Windows-based utilisent `\`.

Par souci de commodité, Java fourni deux options pour récupérer le séparateur local.

```java
System.out.println(System.getProperty("file.separator"));

System.out.println(java.io.File.separator);
```

Le code suivant crée un objet file et détermine si le chemin qu'il indique existe dans le système de fichier

```java
import java.io.File;

public class FileSample {
    public static void main(String[] args) {
        File file = new File("/home/smith/data/zoo.txt");
        System.out.println(file.exists());
    }
}
```
On peut aussi utiliser un chemin relatif comme ceci
```java
File parent = new File("/home/smith");
File child = new File(parent, "data/zoo.txt");
```

#### Travailler avec un objet `File`
Voici une liste des méthodes de `File` les plus communes

|Nom de la méthode|Description|
|---|---|
|`exists()`|Retourne `true` si le fichier ou dossier existe.|
|`getName()`|Retourne le nom du fichier ou du dossier dénoté par ce chemin.|
|`getAbsolutePath()`|Retourne le chemin absolu de ce *path*.|
|`isDirectory()`|Retourne `true` si le *file* dénoté par ce chemin est un dossier.|
|`isFile()`|Retourne `true` si le *file* dénoté par ce chemin est un fichier.|
|`length()`|Retourne le nombre de *bytes* dans le fichier. Pour des raisons de performance, le système de fichier peut allouer plus de *bytes* sur le disque que le fichier utilise.|
|`lastModified()`|Retourne le nombre de millisecondes depuis la dernière modification.|
|`delete()`|Supprime le fichier ou le dossier. Si le chemin correspond à un dossier alors il doit être vide pour pouvoir être supprimé.|
|`renameTo(File)`|Renomme le fichier dénoté par ce chemin.|
|`mkdir()`|Crée un dossier dénoté par ce chemin.|
|`mkdirs()`|Crée un dossier dénoté par ce chemin en incluant tous les dossier parents non existants.|
|`getParent()`|Retourne le chemin abstrait du dossier parent ou `null` s'il n'a pas de parent nommé.|
|`listFiles()`|Retourne un tableau `File[]` dénotant les fichiers du dossier.|

```java
import java.io.File;

public class ReadFileInformation {
    public static void main(String[] args) {
        File file = new File("C:\\data\\zoo.txt");
        System.out.println("Le fichier existe : " + file.exists());
        
        if(file.exists()) {
            System.out.println("Chemin absolu : "+ file.getAbsolutePath());
            System.out.println("Est un dossier : "+ file.isDirectory());
            System.out.println("Chemin parent : "+ file.getParent());
            
            if(file.isFile()) {
                System.out.println("Taille du fichier : "+ file.length());
                System.out.println("Dernière modification : "+ file.lastModified());
            } else {
                for(File subFile: file.listFiles()) {
                    System.out.println("\t" + subFile.getName());
                }
            }
        }
    }
}
```
Si le chemin fourni ne pointe pas vers un *file* :
```
Le fichier existe : false
```

Si le chemin fourni pointe vers un fichier valide : 

```
Le fichier existe : true
Chemin absolu : C:\data\zoo.txt
Est un dossier : false
Chemin parent : C:\data
Taille du fichier : 12382
Dernière modification : 1420070400000
```

Si le chemin fourni pointe vers un dossier valide : 
```
Le fichier existe : true
Chemin absolu : C:\data
Est un dossier : true
Chemin parent : C:\
    employees.txt
    zoo.txt
    zoo-backup.txt
```

Remarquez que pour les chemins Windows-based il faut échapper le backslash.

## Introduction aux streams
Dans cette section, nous présentons le concept de streams en Java et comment is sont utilisés pour le traitement d'entrée/sortie (input/ouput, I/O).

I/O se réfère à la nature de comment les données sont consultées, soit par la lecture des données d'une ressource (input), soit par l'écriture de données dans une ressource (output).

Notez que les streams I/O n'ont rien à voir avec l'API Streams

### Fondamentaux des streams
Le contenu dun fichier peut être lu ou écrit grâce à un stream, qui est une liste d'éléments de donnée présenté séquentiellement. 

Un stream doit être considéré comme un long, presque sans-fin, « courant d'eau » présenté chaque « vague » à la fois.

Chaque type de stream segmente les « vagues » ou « blocs » de sa manière. Certains streams lisent et écrivent un byte à la fois, ou d'autres lisent et écrivent caractère par caractère ou par chaîne de caractère.

De plus, certaines classes de stream écrivent par groupe de byte ou caractères à la fois, spécifiquement celles qui se contenant `Buffered` dans leur nom.

> Tous les streams Java utilisent des Bytes, bien que l'API `java.io` puisse gérer les caractères, les chaînes de caractères ou les groupes de bytes, elles sont toutes basées sur le fait de pouvoir lire ou écrire un byte ou un `array` de byte.

Bien que les streams soient communément utilisés avec *file I/O*, ils sont plus généralement utilisés pour gérer la lecture / l'écriture de source de données séquentielles.

Java fournit trois streams prédéfinis ; `System.in`, `System.out`, `System.err` que nous détaillerons plus tard dans ce chapitre.

### Nomenclature des streams
#### *Byte streams* vs. *Character streams*
L'API `java.io` définit deux sets de classe permettant de lire et d'écrire des streams, ceux avec `Stream` et ceux avec `Reader`/`Writer` dans leurs noms.

Différences entre `Streams` et `Reader`/`Writer` :
1. Les classes *streams* sont utilisées pour entrer ou sortir tout types de données binaires ou byte.
2. Les classes *readers* et *writer* sont utlisées pour entrer ou sortir seulement des caractères ou des données `String`.

Il est important de retenir que même si les *readers*/*writers* ne contiennent pas `Stream` dans leur nom de classe ce sont des streams.

> Pourquoi utiliser des *character streams* ? Par confort.

L'API `java.io` est structurée telle que les classes streams ont le mot `InputStream` ou `OutputStream` dans leur nom, ou que toutes les classes `Reader`/`Writer` aient soit `Reader` soit `Writer` dans leur nom.

#### `Input` et `Output`
La plupart des classes streams `Input` possèdent leur équivalent `Output` et vice-versa. (eg. `FileOutputStream` / `FileInputStream`)

La plupart des classes streams `Reader` possèdent leur équivalent `Writer` et vice-versa. (eg. `FileReader`, `FileWriter`)

Il existe des exceptions à ces règles, Pour l'examen vous devez savoir que `PrintWriter` et `PrintStream` n'ont pas d'équivalent.

### Streams de bas-niveau vs streams de haut-niveau
Une autre façon de vous familiariser avec l'API `java.io` est de diviser les streams en bas-niveau ou haut-niveau.

Un stream de *bas-niveau* se connecte directement avec la source de donnée, comme un fichier, un `array`, ou une `String`. Les streams de bas-niveau traitent les données brute ou les ressources et sont accédées directement et sans filtrage.

Un stream de *haut-niveau* est basé sur un autre stream en utilisant le *wrapping*. Le *wrapping* est le processus par lequel une instance est passée par le constructeur d'une autre classe et par lequel les opérations sont filtrées et appliquées sur l'instance originelle.

```java
// Exemple de stream de haut-niveau avec un stream de bas-niveau en entrée
try (BufferedReader bufferedReader = new BufferedReader(new FileReader("zoo-data.txt"))){
    System.out.println(bufferedReader.readLine());
}
```
Dans cet exemple, le `FileReader` est le stream de bas niveau, le `BufferedReader` est le stream de haut-niveau qui prend le `FileReader` comme argument.

Plusieurs opérations du stream de haut-niveau seront passées au stream de bas-niveau sous-jacent, comme `read()` et `close()`.

Plusieurs opérations surchargent ou ajoutent des fonctionnalités au stream de bas-niveau.

Les stream de haut-niveau ajoute des nouvelles méthodes, comme `readLine()`, et améliore aussi les performances pour la lecture et le filtrage des données de bas-niveau.

Des streams de haut-niveau peuvent prendre d'autres streams de haut niveau en entrée
```java
// Exemple de stream de haut-niveau avec un stream de haut-niveau en entrée 
try(ObjectInputStream objectStream = new ObjectInputStream(
                                                new BufferedInputStream(
                                                        new FileInputStream("zoo-data.txt")))) {
    System.out.println(objectStream.readObject());                                        
}
```

Dans cet exemple, Le `FileInputStream` est les stream de bas-niveau interagissant directement avec le fichier, il est *wrapped* dans le stream de haut-niveau `BufferedInputStream` pour améliorer les performance.

Enfin l'objet entier est *wrapped* dans le stream de haut niveau `ObjectInputStream`, qui permet de filtrer les données comme un objet Java.

Pour l'examen, les seules classes de streams bas-niveau que vous devez connaître sont celles qui opèrent sur les fichiers. Le reste des classes streams non-abstraites sont toutes des streams de haut-niveau.

> En pratique et à moins que vous fassiez quelque chose de vraiment spécialisé dans votre application, vous devriez toujours *wrap* vos *file streams* dans une classe `Buffered`.

#### Classes de base de stream
La librairie `java.io` définit 4 classes abstraites qui sont parentes de toutes les classes définies dans l'API :
- `InputStream`
- `OutputStream`
- `Reader`
- `Writer`

Pour notre confort, les créateurs de l'API incluent le nom de la classe parente comme un suffixe de la classe enfant.

Bien que la plupart des classes de `java.io` suivent ce schéma, `PrintStream`, qui est un `OutputStream`, ne le suit pas.

Les constructeurs des streams de haut-niveau prennent en entrée des références à ces classes.

En utilisant le parent abstrait en entrée, les streams de haut-niveau peuvent être plus facilement utilisés sans se préoccuper de la sous-classe (eg. *wrappings* multiples, *user-defined* streams).

##### Pièges
À l'examen, il faudra savoir déterminer quels classes de streams sont compatibles les unes avec les autres.

```
new BufferedInputStream(new FileReader("zoo-data.txt"));        // ❌ Ne compile pas - mélange Reader/Writer avec InputStream
new BufferedWriter(new FileOutputStream("zoo-data.txt"));       // ❌ Ne compile pas - mélange Reader/Writer avec OutputStream
new ObjectInputStream(new FileOutputStream("zoo-data.txt"));    // ❌ Ne compile pas - mélange InputStream avec OutputStream
new BufferedInputStream(new InputStream("zoo-data.txt"));       // ❌ Ne compile pas - On ne peut pas instancier InputStream (classe abstraite)
```

#### Décoder le nom des classes de Java I/O
- Une classe avec le mot `InputStream` ou `OutputStream` dans son nom est souvent utilisée pour respectivement lire ou écrire des données binaires,
- Un classe avec le mot `Reader` ou `Writer` dans son nom est utilisée pour respectivement lire ou écrire des caractères ou des chaines de caractère,
- La plupart, mais pas toutes, des classes *input* ont leur équivalent *output*,
- Un stream de bas-niveau se connecte directement à la source de données,
- Un stream de haut-niveau se construit au-dessus d'un autre stream en utilisant le *wrapping*,
- Une classe avec `Buffered` dans son nom lit et écrit les données par groupes de bytes ou de caractères, améliorant ainsi les performances dans un *file system* séquentiel.


###  Classes stream de `java.io`
|Nom de la classe|Bas/Haut niveau|Description|
|---|:---:|---|
|`InputStream`|N/A|La classe abstraite à partir de laquelle toutes les classes `InputStream` héritent.|
|`OutputStream`|N/A|La classe abstraite à partir de laquelle toutes les classes `OutputStream` héritent.|
|`Reader`|N/A|La classe abstraite à partir de laquelle toutes les classes `Reader` héritent.|
|`Writer`|N/A|La classe abstraite à partir de laquelle toutes les classes `Writer` héritent.|
|`FileInputStream`|Bas|Lit les données du fichier comme de bytes.|
|`FileOutputStream`|Bas|Écrit les données du fichier comme de bytes.|
|`FileReader`|Bas|Lit les données du fichier comme des caractères.|
|`FileWriter`|Bas|Écrit les données du fichier comme des caractères.|
|`BufferedReader`|Haut|Lit les données de types caractères d'un `Reader` existant de façon *buffered*, ce qui améliore l'efficience et les performances.|
|`BufferedWriter`|Haut|Écrit les données de types caractères d'un `Reader` existant de façon *buffered*, ce qui améliore l'efficience et les performances.|
|`ObjectInputStream`|Haut|Désérialise les types de données Java et graphes de Java d'un `InputStream` existant.|
|`ObjectOutputStream`|Haut|Sérialise les types de données Java et graphes de Java d'un `OutputStream` existant.|
|`InputStreamReader`|Haut|Lit les données de type caractère d'un `InputStream` existant.|
|`OutputStreamWriter`|Haut|Écrit les données de type caractère d'un `OutputStream` existant.|
|`PrintStream`|Haut|Lit une représentation formatée d'objets Java dans un stream binaire.|
|`PrintWriter`|Haut|Écrit une représentation formatée d'objets Java dans un stream *text-based*.|

### Opérations de streams communes
#### Fermer un stream
Puisque les streams sont considérés comme des ressources, il est impératif qu'ils soient fermés une fois qu'ils ont été utilisés pour éviter les fuites de ressources.

Nous pouvons accomplir cela facilement en appelant la méthode `close()` dans un bloc `finally` ou en utilisant la syntaxe *try-with-resources*.

Dans un système de fichier, en cas d'omission de la fermeture du fichier, le fichier peut rester « bloqué » par le système d'exploitation, empêchant à d'autres processus de lire / écrire dessus jusqu'à ce que le programme se termine.

#### *Flush* du stream
Lorsque des données sont écrites vers un `OutputStream`, le système d'exploitation ne garantie pas que les données vont arriver au fichier immédiatement.

Dans beaucoup de systèmes d'exploitation les données sont mis en cache en mémoire, avec l'écriture ne se produisant qu'une fois que le cache est rempli ou qu'un certain délai ne se soit passé.

Si les données sont mises en cache et que l'application s'interrompt de manière inattendue, les données seront perdues car elles n'ont jamais été écrites dans le système de fichier.

Pour contrecarrer cela, Java fournit une méthode `flush()` qui demande à ce que toutes les données accumulées soient écrites immédiatement sur le disque.

La méthode `flush()` aide à réduire le montant de données perdues si l'application se termine de façon inattendue. Cela a un coût, surtout pour les gros fichiers, à chaque fois que vous appelez cela peut créer un ralentissement visible dans votre application.

À moins que les données que vous écrivez ne soient extrêmement sensibles, la méthode `flush()` doit être appelée par intermittence.

Vous n'avez pas besoin d'appeler la méthode `flush()` explicitement si vous avez fini d'écrire dans un fichier puisque la méthode `close()` le fera automatiquement.

#### *Marking* du stream
Les classes `InputStream` et `Reader` incluent les méthodes `mark(int)` et `reset()` pour bouger le stream vers une position antérieure.

Avant d'utiliser ces méthodes, vous devez appeler la méthode `markSupported()` qui retourne `true` si `mark()` est supportée.

Tous les *input streams* de `java.io` ne supportent pas cette opération et tenter d'appeler `mark(int)` ou `reset()` sur une classe qui ne supporte pas cette opération lèvera une exception à l'exécution.

Une fois que vous avez vérifié que le stream supporte ces opérations, vous pouvez appeler `mark(int)` avec une valeur limite de *read-ahead*. Vous pourrez ainsi lire autant de bytes que vous le souhaitez en avance jusqu'à la valeur donnée.

Si à un moment donné vous souhaitez retourner au point où vous avez appelé `mark()`, alors vous avez juste à appeler la méthode `reset()` et le stream reviendra à l'état précédent.

En pratique, ce n'est pas remettre les données dans le stream mais plutôt stocker les données en avance pour pouvoir les relire. De ce fait vous ne devriez pas appeler la méthode `mark()` avec une valeur trop grande car cela pourrait prendre beaucoup de mémoire.

Supposons que les prochaines valeurs du stream soient `ABCD`
```java
InputStream is = ...
System.out.print((char) is.read());
if(is.markSupported()) {
    is.mark(100);
    System.out.println((char) is.read());
    System.out.println((char) is.read());
    is.reset();
}
System.out.println((char) is.read());
System.out.println((char) is.read());
System.out.println((char) is.read());
```
```
// Si l'opération mark() est supportée
ABCBCD

// Si l'opération mark() n'est pas supportée
ABCD
```

Notez que sans tenir compte du fait que `mark()` soit supporté ou non, les streams finissent à la même position.

Finalement, si vous appelez reset après que la limite de lecture de `mark()` soit dépassée, une exception *peut* être levée à l'exécution, puisque les positions marquées peuvent être invalidées.

Elles *peuvent* être levées car certaines implémentations utilisent un *buffer* pour permettre de lire des données supplémentaires avant que les marques soient invalidées.

#### *Skipper* des données
Les classes `InputStream` et `Reader` incluent la méthode `skip(long)`, qui saute un certain nombre de bytes.

Elle retourne une valeur `long` qui indique le nombre de *bytes* qui ont vraiment été sautés.

Si la valeur est zéro ou est négative alors aucun *byte* n'a été sauté (eg. fin du stream).

Supposons que les prochaines valeurs du stream soient `TIGER`
```java
InputStream is = ...
System.out.println((char) is.read());
is.skip(2);
is.read();
System.out.println((char) is.read());
System.out.println((char) is.read());
```
```
TRS
```

Vous pouvez voir dans cet exemple qu'appeler `skip()` est équivalent à appeler `read()` est d'ignorer le résultat.

Pour sauter une poignée de *bytes*, ces deux méthodes sont virtuellement identiques.

Pour un très grand nombre de *bytes*, skip sera souvent plus rapide car is utilisent des *arrays* pour lire les données.
