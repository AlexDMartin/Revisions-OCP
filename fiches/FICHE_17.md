## Travailler avec les streams

### Les classes `FileInputStream` et `FileOutputStream`

Les premières classes de stream que nous allons étudier en détail ont les classes de streams pour les fichiers les plus basiques, `FileInputStream`
et `FileOutputStream`.

Elles sont utilisées pour respectivement lire les *bytes* d'un fichier ou écrire des *bytes* dans un fichier.

Ces classes incluent un constructeur qui prend un objet `File` ou une `String` représentant le chemin du fichier.

Les données dans un `FileInputStream` sont souvent accédés par des appels successifs à la méthode `read()` jusqu'à ce que la valeur `-1` soit retournée,
indiquant que la fin du stream - dans ce cas la fin du fichier - ait été atteinte.

Moins communément, vous pouvez aussi choisir d'arréter de lire le stream prématurément en sortant de la boucle (eg. recherche de `String` dans un fichier)

> Lorsque vous lisez une valeur unique d'une instance de `FileInputStream`, la méthode `read()`retourne une valeur primitive `int` plutôt qu'une valeur `byte`.
> Cela lui permet de retourner spécifiquement `-1` quand la fin du fichier est atteinte.
> Par soucis de compatibilité, `FileOutputStream` renvoit aussi une valeur `int` lors de l'écriture d'une valeur unique dans un fichier.

#### `FileInputStream`

La classe `FileInputStream` contient aussi une version surchargée de la méthode `read()` qui prend un pointeur vers un *array* de `byte` où la donnée est
écrite.

La méthode retourne une valeur *integer* indiquant combien de *bytes* ont pu être lus dams l'*array* de `byte`.

Elle est aussi utilisée par les classes *buffered* pour améliorer les performances.


#### `FileOutputStream`

Un objet `FileOutputStream` est accédé en écrivant des *bytes* successifs en utilisant la méthode `write(int)`.

De la même manière que `FileInputStream`, `FileOutputStream` inclut une version surchargée de la méthode `write()` qui permet de passer un *array* de `byte` qui est aussi utilisée par les classes *buffered*.

#### Exemple d'utilisation
```java
import java.io.*;

public class CopyFileSample {
    public static void copy(File source, File destination) throws IOException {
        try (InputStream in = new FileInputStream(source);
            OutputStream out = new FileOutputStream(destination)) {
            
            int b;
            while((b = in.read()) != -1) {
                out.write(b);
            }
        }
    }
    
    public static void main(String[] args) throws IOException {
        File source  = new File("Zoo.class");
        File destination = new File("ZooCopy.class");
        
        copy(source, destination);
    }
}
```
Notez que si le fichier n'existe pas, une `FileNotFoundException` est levée et est une sous-classe de `IOException`.

La performance de ce code peut être améliorée, notamment pour les gros fichiers, en utilisant des *arrays* de `bytes` est des streams *buffered*.

### Les classes `BufferedInputStream` et `BufferedOutputStream`
On peut renforcer notre implémentation en *wrappant* les `FileInputStream` et `FileOutputStream` avec respectivement `BufferedInputStream` et `BufferedOutputStream`.

Au lieu de lire les données un `byte` à la fois, on utilise la méthode `read(byte[])` de `BufferedInputStream` qui retourne le nombre de *bytes* lus dans l'*array* de `byte` fournit.

Le nombre de *bytes* lus est important pour deux raisons :
- Si la valeur retournée est 0, alors nous savons que nous avons atteints la fin du fichier et nous pouvons nous arrêter de lire dans le `BufferedInputStream`,
- La dernière lecture du fichier a des chances de ne remplir que partiellement l'*array* de `byte`, puisqu'il y a peu de chances que la taille du fichier soit un multiple de la taille du *buffer*.

La valeur du dernier appel nous dit combien de *bytes* de l'*array* appartiennent au fichier, les autres doivent être ignorés puisqu'ils appartiennent à un appel précédent.

Les données sont écries depuis le `BufferedOutputStream` avec la méthode `write(byte[], int, int)`, qui prend un *array de `byte`, un offset et une taille.

La valeur de l'offset est le nombres de valeur à sauter avant d'écrire, souvent mise à `0`.

La taille est le nombre de caractères à écrire depuis l'*array* de `byte`.

#### Pourquoi utiliser les classes *buffered*
Bien que nous aurions pu écrire notre ancien exemple en utilisant un *array* de `byte` sans présenter les classes *buffered*, nous avons choisi de les présenter.

En pratique, il est commun d'utiliser les classes *buffered* pour lire et écrire des données avec un *array* de `byte`.

#### Exemple d'utilisation
```java
import java.io.*;

public class CopyBufferFileSample {
    public static void copy(File source, File destination) throws IOException {
        try(
            InputStream in = new BufferedInputStream(new FileInputStream(source));
            OutputStream out = new BufferedOutputStream(new FileOutputStream(destination));
        ) {
        
            byte[] buffer = new byte[1024];
            int lengthRead;
            while((lengthRead = in.read(buffer)) > 0) {
                out.write(buffer, 0, lengthRead);
                out.flush();
            }
        }
    }
}
```

Nous avons aussi ajouté la méthode `flush()` dans la boucle pour être sûrs que les données écrites arrivent bien au disque avant que les données du prochain *buffer soient lues.

##### Réglage de la taille du *buffer*

Il est commun de choisir une taille de *buffer* qui est une puissance de 2.

Si vous n'en spécifiez pas, une valeur par défaut sera utilisée, généralement une puissance de 2 dans la plupart des JVMs.

Peu importe le choix de la taille du buffer, la plus grande amélioration que vous pouvez voir est un passage d'un accès aux fichier *nonbuffered* à *buffered*.

### Les classes `FileReader` et `FileWriter`
Les classes `FileReader` et `FileWriter`, ainsi que leur équivalent *buffered* sont les classes les plus pratiques de l'API `java.io`, du fait que lire et écrire des données **texte** dans un fichier est très commun.

Comme `FileInputStream` et `FileOutputStream`, les classes `FileReader` et `FileWriter` contiennent respectivement les méthodes `read()` et `write()`.

Ces méthodes lisent/écrivent des valeurs `char` plutôt que des valeurs `byte`; mais renvoient un `int` pour que `-1` puisse être retourné.

Les classes `FileReader` et `FileWriter` contiennent aussi d'autres méthodes, comme `close()` et `flush()`, qui fonctionnent de la même manière.

La classe `Writer`, dont hérite `FileWriter`, propose une méthode `write(String)` qui permet qu'une `String` soit directement écrit vers le *stream*.

Utiliser un `FileReader` vous permet aussi d'utiliser `BufferedReader` pour utiliser la méthode `readLine()`.

#### Les classes `BufferedReader` et `BufferedWriter`
```java
import java.io.*;
import java.util.*;

public class CopyTextFileSample {
    public static List<String> readFile(File source) throws IOException {
        List<String> data = new ArrayList<String>();
        try(BufferedReader reader = new BufferedReader(new FileReade(source))) {
            String s;
            while((s = reader.readLine()) != null) {
                data.add(s);
            }
        }
        return data;
    }
    
    public static void writeFile(List<String> data, File destination) throws IOException {
        try(BufferedWriter writer = new BufferedWriter(new FileWriter(destination))) {
            for(String s: data) {
                writer.write(s);
                writer.newLine();
            }
        }
    }
    
    public static void main(Strin[] args) throws IOException {
        File source = new File("Zoo.csv");
        File destination = new File("ZooCopy.csv");
        List<String> data = readFile(source);
        for(String record: data) {
            System.out.println(record);
        }
        writeFile(data, destination);
    }
}
```

Bien que cet exemple et la solution `InputStream`/`OutputStream` peuvent copier le fichier, seule la solution `Reader`/`Writer` nous donne un accès structuré aux données.

Pour faire la même chose avec la solution `InputStream`/`OutputStream` il faudrait détecter toutes les fins de lignes, ce qui peut être fastidieux.

De plus, vous auriez à détecter le *character encoding*.

#### *Character encoding*
Le *character encoding* détermine comment les caractères sont stockés en *bytes* et plus tard comment ces *bytes* ils sont décodés comme des caractères.

Java supporte une grande variété de *character encoding*, comme par exemple : 
- avec un caractère par *byte*, UTF8 ou ASCII,
- avec deux caractères par *byte*, UTF16.

En Java, le *character encoding* peut être spécifié en utilisant la classe `Charset` en passant une valeur « nom » à la méthode statique `Charset.forName()`.

```java
Charset usAsciiCharset = Charset.forName("US-ASCII);
Charset utf8Charset = Charset.forName("UTF-8);
Charset utf16Charset = Charset.forName("UTF-16);
```

> Le point important ici est que, bien que vous puissiez utiliser les `InputStream`/`OutputStream` pour lire des fichiers textes, il est préférable d'utiliser `Reader`/`Writer`.

### Les classes `ObjectInputStream` et `ObjectOutputStream`
Le procédé par lequel nous convertissons des objets en-mémoire à un format stockable est appelé **sérialisation**, avec un procédé inverse appelé **désérialisation**.

> Bien que comprendre la sérialisation soit importante pour utiliser `ObjectInputStream` et `ObjectOutputStream`, Oracle à un passif d'ajouter ou de remettre la sérialisation au programme de l'examen.
> Vérifiez bien la liste des objectifs avant de passer l'examen.

#### L'interface `Serializable`
Pour pouvoir sérialiser des objets grâce à l'API `java.io` ils doivent implémenter l'interface `java.io.Serializable`. 

L'interface `Serializable` est une interface *marker* ou *tagging*, ce qui veut dire qu'elle ne possède aucune méthode.

N'importe quelle méthode peut implémenter l'interface `Serializable` puisqu'elle ne nécessite aucune méthode à implémenter.

Le but d'implémenter l'interface `Serializable` est d'informer les processus que vous avez bien fait en sorte que l'objet soit `Serializable`, ce qui implique que toutes les variables d'instance soient marquées comme `Serializable`.

La plupart des classes intégrées à Java, comme `String`, sont `Serializable`.

> Notez que pour qu'un objet soit `Serializable` il faut que les objets *nested* soient `Serializable`.

Un processus tentant de sérialiser un objet qui n'implémente pas proprement l'interface `Serializable` lèvera une `NotSerializableException`.

Vous pouvez utiliser le mot-clé `transient` sur la référence pour éviter que cet objet ne soit sérialisé et ainsi éviter la `NotSerializableException`.

En plus des objets `transient`, les membres de classes statiques seront aussi ignorés durant le processus de sérialisation et de désérialisation, puisque les variables statiques n'appartiennent pas à une instance.

Si vous avez besoin qu'une information statique soit stockée vous devrez faire une copie vers un objet d'instance et le sérialiser séparément.

##### Pourquoi ne pas marquer toutes les classes comme `Serializable` ?
Il n'y a aucun coût à marquer une classe comme `Serializable`.

La raison pour laquelle nous ne voulons pas que toutes les classe soient marquées comme `Serializable` est que nous voulons informer la JVM qu'il y a des objets que nous ne voulons pas sérialiser.

En particulier des traitements lourds comme les classes `Thread` ou les classes `Stream`.

#### Exemple d'utilisation
```java
import java.io.Serializable;

public class Animal implements Serializable {
    private static final long serialVersionUID = 1L;
    private String name;
    private int age;
    private char type;
    
    public Animal(String name, int age, char type) {
        this.name = name;
        this.age = age;
        this.type = type;
    }
    
    public String getName() { return name; }
    public int getAge() { return age; }
    public char getType() { return type; }
    
    public String toString() {
        return "Animal [name=" + name + ", age=" + age + ", type=" + type + "]"; 
    }
}
```

La seule chose qui est requise pour que la classe `Animal` soit `Serializable` est d'ajouter le `implements Serializable` à la définition de la classe.

Notez que nous avons aussi ajouté une variable appelée `serialVersionUID`. Bien que ce ne soit pas requis, c'est considéré comme une bonne pratique de la définir et de la modifier à chaque fois que vous modifiez la classe.

Le `serialVersionUID` est stocké avec l'objet sérialisé et aide lors du processus de désérialisation. Le processus de sérialisation utilise le `serialVersionUID` pour identifier une version **unique** de la classe.

De cette manière, si une version plus ancienne est rencontrée durant la désérialisation, une exception sera levée.

Certains outils de désérialisation supportent la conversion automatiquement.

##### Maintenir un *Serial UID*

Nous recommandons que vous ne vous fiiez pas au `serialVersionUID` généré par le compilateur Java et déclariez les vôtres pour chaque classe `Serilizable`.

Différentes versions du compilateur Java peuvent avoir différentes implémentations de la génération du `serialVersionUID`.

#### Sérialiser et désérialiser des objets
L'API `java.io` fournit deux classe de streams pour la sérialisation et la désérialisation des objets appelées `ObjectInputStream` et `ObjectOutputStream`.

La classe `ObjectOutputStream` inclut une méthode pour sérialiser un objet vers le stream appelée `void writeObject(Object)`. Si l'objet concerné n'est pas `Serialiable`, ou n'est pas marqué `transient`, une `NotSerializableException` sera levée à l'exécution.

Pour le processus inverse, l'`ObjectInputStream` contient une méthode appelée `readObject()`. Notez que le type de retour de cette méthode est le type générique `java.lang.Object`, indiquant que l'objet aura besoin d'être *cast* à l'exécution pour être utilisé.

```java
import java.io.*;
import java.util.*;

public class ObjectStreamSample {
    public static List<Animal> getAnimals(File dataFile) throws IOException, ClassNotFoundException {
        List<Animal> animals = new ArrayList<Animal>();
        try(ObjectInputStream in = new ObjectInputStream(new BufferedInputStream(new FileInputStream(dataFile)))) {
            while(true) {
              Object object = in.readObject();
              if(object instanceof Animal) animals.add((Animal) object);
            }
        } catch(EOFException e) {
            // Fin du fichier atteinte
        }
        
        return animals;
    }
    
    public static void createAnimalsFile(List<Animal> animals, File dataFile) throws IOException {
        try(ObjectOutputStream out = new ObjectOutputStream(new BufferedOutputStream(new FileOutputStream(dataFile)))) {
            for(Animal animal: animals) {
                out.writeObject(animal);
            }
        }
    }
    
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        List<Animal> animals = new ArrayList<Animal>();
        animals.add(new Animal("Tommy Tiger", 5, 'T'));
        animals.add(new Animal("Peter Penguin", 8, 'P'));
    
        File dataFile = new File("animal.data");
        createAnimalsFile(animals, dataFile);
        System.out.println(getAnimals(dataFile));
    }
    
}
```
```
[Animal [name=Tommy Tiger, age=5, type=T], Animal [name=Peter Penguin, age=8, type=P]]
```

Pour des raisons de performance, on *wrap* les streams de fichier de bas niveau dans un stream *buffered* puis on chaine le résultat dans un stream objet.

Notez que la méthode `readObject()` lève une exception *checked* `ClassNotFoundException` si l'objet désérialisé n'est pas disponible pour le JRE.

Puisque nous lisons des objets, nous ne pouvons pas utiliser la valeur entière `-1` pour déterminer si nous avons fini de lire le fichier. La méthode correcte pour ce faire est *catch* l'`EOFException`.

##### Méthodes de vérification d'`EOF`
Vous pouvez tomber sur du code avec un `InputStream` qui utilise `while(in.available() > 0)` pour vérifier la fin du stream plutôt que de catcher une `EOFException`.

Le problème de cette approche (notifiée dans la JavaDoc) est qu'elle peut renvoyer 0 même s'il y a des *bytes* restants à lire (elle renvoie le nombre de blocs pouvant être lus sans bloquer le prochain appel).

De ce fait la méthode `available()` d'`InputStream` ne doit jamais être utilisée pour checker la fin d'un stream.


> Il est important de checker les null avant la serialisation, dans notre cas on utilise l'opérateur `instanceof`.

#### Comprendre la création d'objet
Pour l'examen, vous devez savoir comment un objet désérialisé est créé.

Lorsque vous désérialisez un objet, le constructeur de la class sérialisé **n'est pas appelé**.

Java appelle le premier constructeur sans argument de la première classe parent non sérialisable, sautant tous les constructeurs de toutes les classes sérialisables.

De plus toutes les variables statiques et initialisations par défaut sont ignorées.

```java
public class Animal implements Serializable {
    private static final long serialVersionUID = 2L
    private transient String name;
    private transient int age = 10;
    private static char type = 'C';
    
    {this.age = 14;}
    
    public Animal() {
        this.name = "Unknown";
        this.age = 12;
        this.type = 'Q';
    }
    
    public Animal(String name, int age, char type) {
        this.name = name;
        this.age = age;
        this.type = type;
    }
    
    // Mêmes méthodes qu'avant
}
```
```
[Animal [name=null, age=0, type=P], Animal [name=null, age=0, type=P]]
```

La JVM initialise ces variables avec leur valeur par défaut, null pour `String` et 0 pour int.

La variable type est statique et le programme affiche donc la dernière variable qui à été utilisée et qui est partagée par toutes les instances.

#### Les classes `PrintStream` et `PrintWriter`
Les classes `PrintStream` et `PrintWriter` sont des classes de haut-niveau qui écrivent des représentations formatées d'objets Java dans un stream de sortie *text-based*.

- La classe `PrintStream` opère sur les instances d'`OutputStream` et écrit les données comme des *bytes*,
- La classe `PrintWriter` opère sur les instances de `Writer` et écrit les données comme des caractères.

Par souci de commodité, ces classes incluent des constructeurs qui permettent d'ouvrir et d'écrire directement dans les fichiers.

De plus, `PrintWriter` à même un constructeur qui prend un `OutputStream` en entrée, vous permettant de *wrapper* un `OutputStream` dans un `PrintWriter`.

Ces classes sont des classes de **comfort** puisque vous pourriez vous en passer pour écrire une primitive de bas-niveau ou un objet directement dans un stream.

Pour l'examen vous devez savoir que `System.out` et `System.err` sont des objets `PrintStream`.

Puisque `PrintStream` hérite d'`OutputStream` et que `PrintWriter` hérite de `Writer`, les deux supportent la méthode `write()` et fournissent une ribambelle de méthodes *print-based*.

Pour l'examen, vous devez connaitre `print()`, `println()`, `format()` et `printf()`.

À l'inverse de la méthode `write()` qui renvoit une exception *checked* `IOException`, les méthodes *print-based* ne lèvent pas d'exception.

Pour le reste de cette section, nous utiliserons un `PrintWriter` dans nos exemples, puisqu'écrire des `String` comme caractères plutôt que comme `byte` est recommandé. Gardez juste en mémoire que ces exemples sont facilement réécrits avec un objet `PrintStream`.

> Pour l'examen, vous devez savoir que la classe `Console` inclut deux méthodes; `format()` et `printf()` qui prennent un vararg optionnel et formatent le résultat.

##### `print()`
La plus basique ds méthode *print-based* est `print()` qui est surchargée par toutes les primitives Java ainsi que les `String` et les `Object`.

En général ces méthodes font un `String.valueOf()` sur l'argument et appellent `write()` sur le stream sous-jacent.

Elle gère le *character encoding* automatiquement. Par exemple, ces deux méthodes sont équivalentes :

```java
PrintWriter out = new PrintWriter("zoo.log");
out.print(5);                   // Méthode de PrintWriter
out.write(String.valueOf(5));   // Méthode de Writer

out.print(2.0);                 // Méthode de PrintWriter
out.write(String.valueOf(2.0)); // Méthode de Writer

Animal animal = new Animal();
out.print(animal);              // Méthode de PrintWriter
out.write(animal == null ? "null" : animal.toString()); // Méthode de Writer
```

Vous devez savoir depuis l'OCA que `valueOf()` appliqué à un objet appelle sa méthode `toString()` ou renvoie `null` si l'objet n'est pas défini.

##### `println()`
Les méthodes suivantes disponibles dans les classes `PrintStream` et `PrintWriter` sont les méthodes `println`, qui sont virtuellement identiques aux méthodes `print()` sauf qu'elles insèrent un saut de ligne après que la valeur soit insérée.

Cette méthode est pratique si le séparateur saut de ligne est dépendant de la JVM.

Pour connaitre le séparateur :
```java
System.getProperty("line.separator");
```

##### `format()` et `printf()`
Les méthodes `format()` de `PrintStream` et `PrintWriter` sont des méthodes pratiques pour formatter directement vers le stream.

Elles prennent une chaîne de caractère, un `Locale` optionnelle, est un set d'arguments.

Pour mettre les développeurs C à l'aise, la méthode `printf()` a été créée, c'est un passe-plat vers `format`.

```java
public PrintWriter format(String format, Object args...)
public PrintWriter printf(String format, Object args...)
```

```java
public PrintWriter format(Locale l,String format,Object... args)
```

##### Exemple d'application avec `PrintWriter`
```java
import java.io.*; 

public class PrintWriterSample {
    public static void main(String[] args) throws IOException {
        File source = new File("zoo.log");
        try(PrintWriter out = new PrintWriter(new BufferedWriter(new FileWriter(source)))) {
            out.print("Today's weather is: ");
            out.println("Sunny");
            out.print("Today's temperature at the zoo is: ");
            out.print(1/3.0);
            out.println('C');
            out.format("It has rained 10.12 inches this year");
            out.println();
            out.pritnf("It may rain 21.2 more inches this year");
        }
    }
}
```
```
Today's weather is: Sunny
Today's temperature at the zoo is: 0.33333333333333333333C
It has rained 10.12 inches this year
It may rain 21.2 more inches this year
```

#### Conclusion sur les classes *stream*
<table>
<thead>
  <tr>
    <th rowspan="3"><code>InputStream</code></th>
    <th colspan="2"><code>FileInputStream</code></th>
  </tr>
  <tr>
    <td><code>FilterInputStream</code></td>
    <td><code>BufferedInputStream</code></td>
  </tr>
  <tr>
    <td colspan="2"><code>ObjectInputStream</code></td>
  </tr>
</thead>
<tbody>
  <tr>
    <td rowspan="2"><code>Reader</code></td>
    <td colspan="2"><code>BufferedReader</code></td>
  </tr>
  <tr>
    <td><code>InputStreamReader</code></td>
    <td><code>FileReader</code></td>
  </tr>
  <tr>
    <td rowspan="4"><code>OutputStream</code></td>
    <td colspan="2"><code>FileOutputStream</code></td>
  </tr>
  <tr>
    <td rowspan="2"><code>FilterOutputStream</code></td>
    <td><code>BufferedOutputStream</code></td>
  </tr>
  <tr>
    <td><code>PrintStream</code></td>
  </tr>
  <tr>
    <td colspan="2"><code>ObjectOutputStream</code></td>
  </tr>
  <tr>
    <td rowspan="3"><code>Writer</code></td>
    <td colspan="2"><code>BufferedWriter</code></td>
  </tr>
  <tr>
    <td><code>OutputStreamWriter</code></td>
    <td><code>FileWriter</code></td>
  </tr>
  <tr>
    <td colspan="2"><code>PrintWriter</code></td>
  </tr>
</tbody>
</table>

##### Autres classes *stream*
Ces classes ne sont pas à l'examen :
- `InputStreamReader` prend une instance d'`InputStream` et renvoie un objet `Reader`,
- `OutputStreamWriter` prend une instance d'`OutputStream` et renvoie un objet `Writer`,
- `DataInputStream` et `DataOutputStream`fonctionnent comme les classes de stream objet sauf qu'elles n'écrivent que des primitives et nécessitent que les données soient lues dans un certain ordre (= trop rigide),
- `FilterInputStream` et `FilterOutputStream` sont les super-classes de toutes les classes qui filtrent ou transforment les données.

## Interagir avec les utilisateurs
La dernière classe de `java.io` que nous allons présenter est `java.io.Console`.

La classe `Console` a été ajoutée en java 6 pour avoir une form plus évoluée des classes *stream* `System.in` et `System.out`.

C'est maintenant la technique recommandée pour interagir avec et afficher des informations à l'utilisateur dans un environnement *text-based*.

### L'ancienne manière
De la même manière que `System.out` retourne un `OutputStream`, `System.in` renvoi un `InputStream` qui est utilisé pour récupérer la saisie de l'utilisateur.

Il peut être enchaîné avec un `BufferedReader` pour que la saisie soit terminée avec la touche entrée.

Avant que nous puissions appliquer le `BufferedReader`, nous avons besoin de *wrap* `System.in` dans la classe `InputStreamReader` qui nous permet de créer un objet `Reader` depuis une instance d'`InputStream` existante.

```java
import java.io.*;

public class SystemInSample {
    public static void main(String[] args) throws IOException {
        BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        
        String userInput = reader.readLine();
        System.out.println("You entered the following: " + userInput);
    }
}
```

Remarquez que nous n'avons pas fermé le stream, fermer System.in empêcherait notre application d'accepter une saisie de l'utilisateur.

### La nouvelle manière
La classe `Console` est un singleton.

Elle est accédée par la méthode `System.console()` (Celle-ci renverra `null` dans un environnement où les interactions par texte ne sont pas permises).

```java
import java.io.Console;

public class ConsoleSample {
    public static void main(String[] args) {
        Console console = System.console();
        if(console != null) {
          String userInput = console.readLine();
          console.writer()
                 .println("You entered the following: " + userInput);
        }
    }
}
```

Voici les méthodes de `Console` que vous devez connaître pour l'examen :
#### `reader()` et `writer()`
Les méthodes `reader()` et `writer()` permettent respectivement d'accéder à l'instance de `Reader` et `PrintWriter`.

Elles sont similaires à accéder à `System.in` et à `System.out` directement.

Cette manière permet de gérer le *character encoding* directement.

#### `format()` et `printf()`
La classe `Console` définit une seule méthode `format()` qui ne prend pas de `Locale` en paramètre, elle prend donc la locale par défaut.

Bien sûr vous pouvez passer votre locale en récupérant l'objet `Writer` comme ceci :
```java
Console console = System.console();
console.writer.format(new Locale("fr", "CA"), "Hello world!");
```

```java
// Exemple d'affichage avec la classe Console
import java.io.*;

public class ConsoleSamplePrint {
    public static void main(String[] args) throws NumberFormatException, IOException {
        Console console = System.console();
        if(console == null) {
            throw new RuntimeException("Console not available");
        } else {
            console.writer().println("Welcome to out zoo!");
            console.format("Our zoo has 391 animals and employs 25 people.");
            console.writer().println();
            console.printf("The zoo spans 128.91 acres.");
        }
    }
}
```

#### `flush()`
La méthode `flush()` force les données *buffered* à être écrites directement.

Il est recommandé que vous appeliez la méthode `flush()` avant d'appeler les méthodes `readLine()` ou `readPassword()` pour être sûr qu'aucune donnée ne soit en attente pendant la lecture.

#### `readLine()`
La méthode `readLine()` récupère une seule ligne de texte de l'utilisateur, l'utilisateur la termine en appuyant sur la touche entrée.

La classe `Console` supporte aussi une version surchargée de `readLine()` qui affiche une demande formatée avant d'afficher le texte
```java
public String readLine(String format, Object... args)
```
```java
// Exemple d'utilisation de readLine()
import java.io.*;

public class ConsoleReadInputSample {
    public static void main(String[] args) throws NumberFormatException, IOException {
        Console console = System.console();
        if(console == null) {
            throw new RuntimeException("Console not available");
        } else {
            console.writer().print("How excited are you about your trip today? ");
            console.flush();
            String excitementAnswer = console.readLine();
            String name = console.readLine("Please enter your name: ");
            Integer age = null;
            console.writer().print("What is your age? ");
            console.flush();
            BufferedReader reader = new BufferedReader(console.reader());
            String value = reader.readLine();
            age = Integer.valueOf(value);
            console.writer().println();
            
            console.format("Your name is " + name);
            console.writer.println();
            console.format("Your age is " + age);
            console.printf("Your excitement level is: " + excitementAnswer);
        }
    }
}
```

#### `readPassword()`
La méthode `readPassword` est similaire à la méthode `readLine()` sauf que l'*echoing* est désactivé. De ce fait l'utilisateur ne voit pas ce qu'il tape, ce qui veut dire que leurs mots de passe sont sécurisés même si quelqu'un regarde leur écran.

Comme pour `readLine()`, la classe `Console` supporte aussi une version surchargée de `readPassword()` qui affiche une demande formatée avant d'afficher le texte.

```java
public char[] readPassword(String format, Object... args)
```

> Pourquoi un tableau de caractères plutôt qu'une `String` ?
> Comme les `String` sont stockées dans le string pool, on pourrait les récupérer malicieusement.

```java
// Exemple d'utilisation de readPassword()
import java.io.*;
import java.util.Arrays;

public class PasswordCompareSample {
    public static void main(String[] args) throws NumberFormatException, IOException {
        Console console = System.console();
        if(console == null) {
            throw new RuntimeException("Console not available");
        } else {
            char[] password = console.readPassword("Enter your password: ");
            console.format("Enter your password again: ");
            console.flush();
            char[] verify = console.readPassword();
            
            boolean match = Arrays.equals(password, verify);
            
            // Effacement des mots de passes de la mémoire
            for(int i = 0; i < password.length; i++) {
                password[i] = 'x';
            }
            for(int i = 0; i < verify.length; i++) {
                password[i] = 'x';
            }
            
            console.format("Your password was "+ (match ? "correct" : "incorrect"));
        }
    }
}
```