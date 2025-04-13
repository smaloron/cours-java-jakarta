# JPQL (Jakarta Persistence Query Language)

JPQL est un langage de requête défini par la spécification JPA (Java Persistence API) qui permet de gérer la persistance
des données dans les applications Java. Ce langage utilise une syntaxe proche de SQL, mais contrairement à ce dernier
qui interroge directement les tables d'une base de données, JPQL effectue des requêtes sur des entités JPA, qui
représentent les objets mappés dans des tables de bases de données relationnelles.

La caractéristique fondamentale de JPQL est son indépendance vis-à-vis du stockage de données sous-jacent, ce qui le
rend portable d'une base de données à une autre. Comme l'indique la documentation Oracle : "Le langage de requête Java
Persistence définit des requêtes pour les entités et leur état persistant. Ce langage permet d'écrire des requêtes
portables qui fonctionnent indépendamment du magasin de données sous-jacent".

JPQL s'inscrit dans une approche ORM (Object-Relational Mapping) qui vise à résoudre l'incompatibilité entre le modèle
objet d'une application Java et le modèle relationnel d'une base de données. JPA, avec JPQL, permet de convertir
automatiquement et à la demande la base de données sous forme d'un graphe d'objet.

**Exemple de requête JPQL**

```SQL
SELECT u FROM User u
```

**Comparaison avec SQL**
JPQL ressemble à SQL dans sa syntaxe, mais présente des différences fondamentales :

- SQL opère sur des tables, des lignes et des colonnes, tandis que JPQL opère sur des entités et leurs attributs.

- JPQL est indépendant de la base de données utilisée, alors que SQL peut varier selon les systèmes de gestion de 
bases de données.

- Les requêtes JPQL dépendent des entités JPA définies dans l'application et non des structures de tables dans la 
base de données.

## La syntaxe de JPQL

### Les trois types de requêtes JPQL

JPQL permet de réaliser ces trois opérations :

- Une instruction SELECT pour récupérer des données

- Une instruction UPDATE pour mettre à jour des données

- Une instruction DELETE pour supprimer des données

L'instruction `INSERT INTO` est absente, car cette opération est gérée par la méthode `persist` de `EntityManager`.


### SELECT avec JPQL

**Structure de la requête**

```
SELECT [DISTINCT] alias 
FROM Entité [AS] alias 
[WHERE condition]
[ORDER BY attribut [ASC|DESC]]
[GROUP BY propriété]
[HAVING condition]
```

#### Les opérateurs conditionnels


| **Catégorie**                 | **Opérateur**                | **Description**                                                            | **Exemple**                                                                                  |
|:------------------------------|:-----------------------------|:---------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------|
| **Opérateurs de comparaison** | =, <>, <, <=, >, >=          | Comparaison d'égalité, d'inégalité, ou de valeurs supérieures/inférieures. | `x.age &gt;= 18` (âge supérieur ou égal à 18)                                                |
|                               | IS NULL, IS NOT NULL         | Vérifie si une valeur est nulle ou non nulle.                              | `x.name IS NOT NULL` (le nom n'est pas nul)                                                  |
|                               | BETWEEN, NOT BETWEEN         | Vérifie si une valeur est dans une plage donnée.                           | `x.age BETWEEN 18 AND 30` (âge entre 18 et 30 inclus)                                        |
|                               | IN, NOT IN                   | Vérifie si une valeur appartient ou non à une liste.                       | `x.name IN ('Alice', 'Bob')` (nom parmi Alice ou Bob)                                        |
|                               | LIKE, NOT LIKE               | Recherche de correspondances avec des motifs (wildcards : `%`, `_`).       | `x.name LIKE 'A%'` (nom commençant par "A")                                                  |
| **Opérateurs logiques**       | AND                          | Combine deux conditions qui doivent toutes les deux être vraies.           | `x.age &gt; 18 AND x.city = 'Paris'`                                                         |
|                               | OR                           | Combine deux conditions dont au moins une doit être vraie.                 | `x.age &lt; 18 OR x.city = 'Paris'`                                                          |
|                               | NOT                          | Inverse la condition.                                                      | `NOT x.active` (non actif)                                                                   |
| **Opérateurs de collection**  | IS EMPTY, IS NOT EMPTY       | Vérifie si une collection est vide ou non.                                 | `x.orders IS EMPTY` (aucune commande associée)                                               |
|                               | MEMBER [OF], NOT MEMBER [OF] | Vérifie si un élément fait partie ou non d'une collection.                 | `'Doe' MEMBER OF employee.name`                                                              |
| **Autres opérateurs**         | EXISTS, NOT EXISTS           | Vérifie la présence de résultats dans une sous-requête.                    | `EXISTS (SELECT o FROM Order o WHERE o.customer = x)`                                        |
|                               | ALL, ANY, SOME               | Compare une valeur avec tous ou certains résultats d'une sous-requête.     | `x.salary &gt;= ALL(SELECT e.salary FROM Employee e)` (salaire ≥ à tous les autres salaires) |

#### Les fonctions JPQL

| **Catégorie**                 | **Fonction**                     | **Description**                                              | **Exemple**                            |
|:------------------------------|:---------------------------------|:-------------------------------------------------------------|:---------------------------------------|
| **Fonctions de chaîne**       | CONCAT(string1, string2)         | Concatène deux chaînes de caractères.                        | `CONCAT(x.firstName, ' ', x.lastName)` |
|                               | SUBSTRING(string, start, length) | Extrait une sous-chaîne à partir d'une chaîne donnée.        | `SUBSTRING(x.name, 1, 5)`              |
|                               | TRIM(string)                     | Supprime les espaces en début et fin de chaîne.              | `TRIM(x.name)`                         |
|                               | LOWER(string)                    | Transforme une chaîne en minuscules.                         | `LOWER(x.name)`                        |
|                               | UPPER(string)                    | Transforme une chaîne en majuscules.                         | `UPPER(x.name)`                        |
|                               | LENGTH(string)                   | Retourne la longueur d'une chaîne.                           | `LENGTH(x.name)`                       |
|                               | LOCATE(search, string, offset)   | Trouve la position d'une sous-chaîne dans une chaîne donnée. | `LOCATE('test', x.description, 1)`     |
| **Fonctions numériques**      | ABS(number)                      | Retourne la valeur absolue d'un nombre.                      | `ABS(x.salary)`                        |
|                               | SQRT(number)                     | Retourne la racine carrée d'un nombre.                       | `SQRT(x.salary)`                       |
|                               | MOD(dividend, divisor)           | Retourne le reste d'une division.                            | `MOD(x.salary, 10)`                    |
| **Fonctions de date/temps**   | CURRENT_DATE()                   | Retourne la date actuelle.                                   | `CURRENT_DATE()`                       |
|                               | CURRENT_TIME()                   | Retourne l'heure actuelle.                                   | `CURRENT_TIME()`                       |
|                               | CURRENT_TIMESTAMP()              | Retourne la date et l'heure actuelles.                       | `CURRENT_TIMESTAMP()`                  |
| **Fonctions spécifiques JPA** | SIZE(collection)                 | Retourne la taille d'une collection associée à une entité.   | `SIZE(x.orders)`                       |
|                               | INDEX(orderedCollection)         | Retourne l'index d'un élément dans une collection ordonnée.  | `INDEX(x.items)`                       |
|                               | TREAT(entity AS Type)            | Effectue un downcast vers un type spécifique.                | `TREAT(x AS SubType)`                  |

## Exécution des requêtes JPQL

Pour exécuter une requête JPQL, il faut passer par `EntityManager`.
La méthode `createQuery` de cette classe retourne un objet `Query` ou `TypedQuery` et admet deux arguments :

- Une chaîne représentant la requête JPQL
- Une chaîne représentant l'entité cible. Ici, il sera plus pratique d'utiliser la notation `Entity.class` ce qui 
  évitera d'avoir à indiquer le nom du package.

### Requête sans paramètres

**Avec `TypedQuery`**
```Java
// Où em est une instance de l'EntityManager
TypedQuery<Product> query = em.createQuery(
    "SELECT p FROM Product p", Product.class);
List<Product> productList = query.getResultList();

Product firstProduct = productList.get(0);
```

**Avec `Query`**
```Java
Query query = em.createQuery(
    "SELECT p FROM Product p", Product.class);
List<Object> productList = query.getResultList();

// Il faut faire un cast car la requête n'est pas typée
Product firstProduct = (Product) productList.get(0);
```

**Un exemple complet**

```java
import jakarta.persistence.*;
import fr.mvc.app.model.entity.User;


public class JPQLSelectUserExample {
    
    public static EntityManager getManager(){
        EntityManagerFactory emf;
        // Création de la fabrique d'EntityManager
        // L'argument correspond au nom de l'unité de persistence
        // tel que définit dans persistence.xml
        emf = Persistence.createEntityManagerFactory(
                "MyPersistenceUnit"
        );

        // Création de l'EntityManager avec la fabrique
        return emf.createEntityManager();
    }
    
    public static void main(String[] args) {
        
        EntityManager em = getManager();

        // Définition de la requête
        TypedQuery<User> query = em.createQuery(
                "SELECT u FROM User u", User.class);
        
        // Exécution de la requête
        List<User> produits = query.getResultList();

        // Affichage des utilisateurs
        if(users != null && !users.isEmpty()) {
            for(User user : users) {
                System.out.println("Utilisateur: " + user.getUsername());
                System.out.println("Email: " + user.getEmail());
                System.out.println("---------------------");
            }
        } else {
            System.out.println("Aucun utilisateur trouvé");
        }

    }
}
```

### Requête paramétrée

Pour les requêtes admettant des paramètres, il faudra utiliser des marqueurs et indiquer leur valeur avec la méthode 
`setParameter()`.

```java
import jakarta.persistence.*;
import fr.mvc.app.model.entity.User;


public class JPQLSelectWithParamExample {
    
    public static EntityManager getManager(){
        EntityManagerFactory emf;
        // Création de la fabrique d'EntityManager
        // L'argument correspond au nom de l'unité de persistence
        // tel que définit dans persistence.xml
        emf = Persistence.createEntityManagerFactory(
                "MyPersistenceUnit"
        );

        // Création de l'EntityManager avec la fabrique
        return emf.createEntityManager();
    }
    
    public static void main(String[] args) {
        
        EntityManager em = getManager();

        // 1. Création de la requête avec paramètre nommé
        String jpql = "SELECT u FROM User u WHERE u.email = :email";
        TypedQuery<User> query = em.createQuery(jpql, User.class);

        // 2. Définition de la valeur du paramètre
        String searchedEmail = "moi@moi.com";
        query.setParameter("email", searchedEmail);

        // 3. Exécution et récupération du résultat
        try {
            User user = query.getSingleResult();
            System.out.println("Utilisateur trouvé : " + user.getUsername());
        } catch (NoResultException e) {
            System.out.println("Aucun utilisateur avec cet email");
        } catch (NonUniqueResultException e) {
            System.out.println("Plusieurs utilisateurs avec cet email");
        }
    }
}
```

#### Un exemple de mise à jour

Même principe pour la mise à jour excepté qu'au lieu de `getResultList()` ou `getSingleResult()`, il faut utiliser 
la méthode `executeUpdate()` pour exécuter la requête. Cette méthode retourne le nombre de lignes affectées.


Il est également considéré comme une bonne pratique d'utiliser des transactions pour toutes les opérations qui 
modifient le contenu de la base de données.

```Java
em.getTransaction().begin();
int updated = em.createQuery(
        "UPDATE User u SET u.email = :email WHERE u.id < :id"
        )
        .setParameter("email", "elle@elle.com")
        .setParameter("id", 1)
        .executeUpdate();
em.getTransaction().commit();
```

### Les méthodes de Query

| Méthode             | Description                            | Type de résultat attendu        |
|---------------------|----------------------------------------|---------------------------------|
| `getResultList()`   | Retourne une liste d'objets            | Plusieurs résultats             |
| `getSingleResult()` | Retourne un seul objet                 | Un seul résultat                |
| `executeUpdate()`   | Exécute une requête d'update ou delete | Nombre d'entités affectées      |
| `getFirstResult()`  | Retourne le premier résultat           | Premier objet ou `null` si vide |

### La pagination

Pour paginer les résultats, il faut utiliser les méthodes suivantes :

- `setMaxResults(int nb)` : pour limiter le nombre d'entités retournées.
- `setFirstResult(int position)` : pour indiquer le décalage (offset) du premier résultat de la liste.

**Un exemple de pagination**

```java
int pageNumber = 2;
int pageSize = 10;

String jpql = "SELECT p FROM Product p ORDER BY p.id";
TypedQuery<Product> query = em.createQuery(jpql, Product.class);

query.setFirstResult((pageNumber - 1) * pageSize);
query.setMaxResults(pageSize);

List<Product> products = query.getResultList();
```