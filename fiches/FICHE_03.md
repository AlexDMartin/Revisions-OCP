## Travailler avec les patrons de conception
Un patron de conception est une solution générale établie pour les problèmes de développement communs.

Ils permettent de donner un vocabulaire commun à tous les développeurs.

Ce chapitre sera concentré sur les patrons de conception de création, quatre d'entre eux seront présentés mais seuls le *singleton* et l'*objet immutable* sont a l'examen.

### Appliquer le patron *Singleton*
Problème : Comment créer un unique objet en memoire par application et le partager entre plusieurs classes ?

Solution :  
- le patron singleton est un patron de création qui crée une **unique** instance en mémoire et la partage avec toutes les classes et tous les threads de l'application,
- les singletons peuvent améliorer les performances en chargeant des données réutilisables qui prendraient trop de temps a être stockées et rafraîchies à chaque fois qu'on en aurait besoin,
- on y accède par une seule méthode `public static`, souvent appelée `getInstance`,
- tous les constructeurs d'un singleton sont `private`, En faisant comme cela nous marquons la classe comme `final`. La classe singleton est *effectively final*.

#### Instanciation statique
```java
public class LlamaTrainer {
    public boolean feedLlamas(int numberOfLlamas) {
        int amountNeeded = 5 * numberOfLlamas;
        HayStorage hayStorage = HayStorage.getInstance(); // <-- On récupère l'instance
        if(hayStorage.getHayQuantity < amountNeeded) {
            hayStorage.addHay(amountNeeded + 10);
        }
        boolean fed = hayStorage.removeHay(amountNeeded);
        if(fed) System.out.println("Llamas have been fed");
        return fed;
    }
}
```

#### Instanciation via **static initializer**
```java
// Instantiation using a static block
public class StaffRegister {
    private static final StaffRegister instance;

    static {
        instance = new StaffRegister();
        // Perform additional steps
    }

    private StaffRegister() {}
    
    public static StaffRegister getInstance() {
        return instance;
    }

    // Data access methods

}
```

### Appliquer l'instanciation *lazy* aux Singletons
```java
// Lazy instantiation
public class VisitorTicketTracker {
    private static VisitorTicketTracker instance;

    private VisitorTicketTracker() {}
   
    public static VisitorTicketTracker getInstance() {
        if(instance == null) {
            instance = new VisitorTicketTracker(); // NOT THREAD-SAFE!
        }
        return instance;
    }

    // Data access methods

}
```

### Créer des singletons uniques
```java
// Lazy loaded with synchronized
public class VisitorTicketTracker {
    private static VisitorTicketTracker instance;

    private VisitorTicketTracker() {}
   
    public static synchronized VisitorTicketTracker getInstance() { // <-- ajout du mot-clé « synchronized »
        if(instance == null) {
            instance = new VisitorTicketTracker();
        }
        return instance;
    }

    // Data access methods

}
```
La méthode `getInstance()` est maintenant `synchronized`, ce qui veut dire que seulement un thread sera accepté dans la méthode en même temps.

Ce qui assure qu'un seul objet sera créé.

#### Singletons avec double vérification
```java
// Lazy loaded with synchronized
public class VisitorTicketTracker {
    private static volatile VisitorTicketTracker instance;

    private VisitorTicketTracker() {}
   
    public static VisitorTicketTracker getInstance() { // <-- ajout du mot-clé « synchronized »
        if(instance == null) {
            synchronized (VisitorTicketTracker.class) {
                if(instance == null) {
                    instance = new VisitorTicketTracker(); 
                }
            }
        }
        return instance;
    }

    // Data access methods

}
```

### Créer des objets immutables
Problème : Comment créer des objets en lecture seule qui peuvent être partagés par différentes classes ?

Solution : 
- Le patron objet immutable est un patron de création basé sur l'idée de créer des objets avec un état ne changeant pas une fois qu'ils ont été créés,
- Le patron objet immutable va de paire avec l'encapsulation sauf qu'il n'y a pas de setter,
- Puisque l'état d'un objet immutable ne change jamais, il est intrinsèquement thread-safe.

### Appliquer une stratégie d'immutabilité
Stratégie commune :
1. Utiliser un constructeur pour définir les propriétés de l'objet
2. Marquer les variables d'instance `public` et `final`
3. Ne pas définir de methode « setter »
4. Ne pas permettre de modifier les objets mutables référencés ou d'y accéder directement
5. Empêcher que les méthodes soient redéfinies (*overridden*)

### « Modifier » un objet immutable
On ne peut pas, il faut créer un nouvel objet (une copie de l'objet)

## Utiliser le patron « Builder »
Problème : Comment créer un objet qui requiert plusieurs valeurs d'être définies au moment où l'objet est instancié ?

Solution : 
- le patron *Builder* est un patron de création dans lequel des paramètres sont passés à un objet builder, la plupart du temps via un enchaînement de méthodes, et un objet est généré par un ultime appel a build
- il est souvent utilisé avec des objets immutables puisque les objets immutables n'ont pas de méthodes «setter» et doivent donc être créés avec leur jeu complet de paramètres
- il peut aussi être utilisé avec des objets mutables

Toutes les méthodes « setter » renvoient une instance de l'objet *Builder*.

La classe *builder* et la classe cible sont considérés comme fortement couplées.

Les classes fortement couplées (*tight coupling*) sont extrêmement dépendante l'une de l'autre, un changement mineur dans l'une des classes impact fortement l'autre.

A l'inverse, Des classes faiblement couplées (*loose coupling*) sont faiblement dépendante l'une de l'autre et ont un minimum de dépendances les unes envers les autres.

Même si le faible couplage est préféré dans la pratique, un fort couplage est requis dans ce cas.

Dans la pratique, une classe *builder* est souvent packaged avec sa classe cible, soit en étant une static inner class, soit dans le même package Java.

Un choix que les développeurs de la classe cible peuvent prendre est de faire en sorte que le constructeur soit *privé* ou *default package* pour forceer les utilisateurs à s'appuyer sur le builder pour obtenir une instance de la classe cible.

### Créer des objets avec le patron « Factory »
Problème : Comment écrire du code qui crée des objets dont le type précis ne peut être connu avant l'execution ?

Solution : Le patron « *factory* », appelé aussi « *factory method pattern* », est un patron de création basé sur l'idée d'utiliser une classe *factory* qui produit des instance d'objets par rapport à un jeu de paramètre.

Il est similaire au patron « *builder* », bien qu'il soit centrée sur le support du polymorphisme de classe.
