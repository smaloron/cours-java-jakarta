# Hibernate et JPA

### Définitions

**JPA (Java|Jakarta Persistence API)** est une spécification qui définit un ensemble de règles pour la gestion des
données relationnelles en Java.
Elle fournit des annotations et des interfaces pour mapper des objets Java à des tables de bases de données.

**Hibernate** est une implémentation de JPA (mais peut aussi fonctionner sans JPA).
C’est un ORM (Object-Relational Mapping) qui permet de gérer la persistance des objets Java dans une base de données
relationnelle.

**Pourquoi utiliser Hibernate avec JPA ?**

- Hibernate est plus complet que JPA seul : il offre des fonctionnalités avancées comme la mise en cache, des outils de
  migration, et des critères de requêtes.

- En utilisant JPA avec Hibernate, on bénéficie de la portabilité entre les implémentations tout en tirant parti des
  extensions spécifiques d’Hibernate.

## Installation

### Dépendances

Avec Maven, dans `pom.xml`, il faut déclarer les dépendances suivantes :

- Hibernate
- JPA
- Un pilote JDBC

```xml

<dependencies>
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-core</artifactId>
        <version>6.3.1.Final</version>
    </dependency>
    <dependency>
        <groupId>jakarta.persistence</groupId>
        <artifactId>jakarta.persistence-api</artifactId>
        <version>3.1.0</version>
    </dependency>
    <dependency>
        <groupId>org.mariadb.jdbc</groupId>
        <artifactId>mariadb-java-client</artifactId>
        <version>3.1.4</version>
    </dependency>
</dependencies>

```

### Configuration

La configuration s'effectue dans un fichier `persistence.xml` qui se place dans le chemin suivant :
`src/main/resources/META-INF/persistence.xml`.

> Note : Si Hibernate est utilisé sans JPA, le fichier de configuration se nommera
`hibernate.cfg.xml` et aura une structure différente.

```xml

<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence"
             version="2.2">
    <persistence-unit name="MyPersistenceUnit" transaction-type="RESOURCE_LOCAL">
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        <class>fr.mvc.app.model.entity.User</class>
        <properties>
            <property name="hibernate.connection.driver_class" value="org.mariadb.jdbc.Driver"/>
            <property name="hibernate.connection.url" value="jdbc:mariadb://localhost:3306/mydb"/>
            <property name="hibernate.connection.username" value="root"/>
            <property name="hibernate.connection.password" value="password"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.MariaDBDialect"/>
            <property name="hibernate.hbm2ddl.auto" value="update"/>
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
        </properties>
    </persistence-unit>
</persistence>
```

#### Balise `<persistence-unit>`

Cette balise regroupe les informations nécessaires pour configurer une unité de persistance.
L'unité de persistance regroupe les classes d'entité, les paramètres de connexion à la base de données
et les configurations spécifiques du fournisseur JPA (comme Hibernate) en un seul endroit.
Cela permet à JPA de créer et de gérer des objets EntityManager pour manipuler les données.

Attributs :

- `name` : Nom de l'unité de persistance. Utilisé pour référencer cette unité dans votre code.

- `transaction-type` : Définit le type de transaction :

    - `RESOURCE_LOCAL` : Les transactions sont gérées manuellement (le plus courant pour les applications autonomes).

    - `JTA` : Utilisé pour les applications d'entreprise dans les cas suivants :
        - Quand il faut gérer des transactions impliquant des objets distribués sur de multiples serveurs.
        - Quand la transaction s'effectue sur de multiples SGBDR.
        - Quand la transaction doit inclure l'utilisation de JMS (Java Message Service).
        - Avec Spring Boot quand on souhaite une gestion automatique des transactions par le biais de l'annotation
          `@Transactional`.

#### Balise `<provider>`

Spécifie le fournisseur JPA.

Pour Hibernate : `org.hibernate.jpa.HibernatePersistenceProvider`

#### Balise `<class>`

Liste les classes d'entités prises en charge par l'unité de persistance.
Chaque entité mappée à une table doit être déclarée dans sa propre balise class.

**Exemple**

```xml

<persistence-unit name="MyPersistenceUnit" transaction-type="RESOURCE_LOCAL">
    <class>com.example.entity.User</class>
    <class>com.example.entity.Product</class>
</persistence-unit>
```

Hibernate propose une fonctionnalité de scan automatique des entités qui évite d'avoir à toutes les déclarer dans la
configuration

```xml

<property name="hibernate.archive.autodetection" value="class"/>
```

#### Balise `<properties>`

Contient les paramètres de configuration Hibernate :

##### Paramètres de connexion

    - `hibernate.connection.driver_class` : Driver JDBC, ici `org.mariadb.jdbc.Driver` pour MariaDB.

    - `hibernate.connection.url` : URL de la base de données.

    - `hibernate.connection.username` et `hibernate.connection.password` : Identifiants pour se connecter à la base.

##### Dialecte

- `hibernate.dialect` : Spécifie le dialecte SQL utilisé pour la base de données. Hibernate générant du code SQL,
  il est important que ce dialecte corresponde au SGBDR utilisé.

##### Gestion des tables

- `hibernate.hbm2ddl.auto` : Gère la génération du schéma :

    - `validate` : Vérifie la conformité du schéma existant.

    - `update` : Met à jour le schéma en fonction des entités.

    - `create` : Crée les tables à chaque démarrage.

    - `create-drop` : Crée et détruit les tables à l'arrêt.

    - `none` : Ne fait rien.

##### logs SQL

- `hibernate.show_sql` : Affiche les requêtes SQL dans la console.

- `hibernate.format_sql` : Formate les requêtes pour une meilleure lisibilité.



### Base de données

```yaml
services:
  mariadb:
    image: mariadb:latest
    container_name: mariadb_container
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: 123
      MYSQL_DATABASE: formation
    ports:
      - "3306:3306"
    volumes:
      - mariadb_data:/var/lib/mysql
    networks:
      - mariadb_network

volumes:
  mariadb_data:

networks:
  mariadb_network:
```

## Création des entités

La correspondance entre le monde objet et le monde relationnel s'effectue via des annotations dans les classes d'entité.
Dans la plupart des cas, il n'est pas nécessaire de définir le type de données. Java étant fortement typé, Hibernate 
pourra inférer le type SQL en fonction du type de la propriété et ce qu'il s'agisse de type primitif ou de wrapper 
class.

L'annotation `@Column` permet de définir les paramètres de la correspondance. 
Il est toutefois possible de l'omettre, dans ce cas Hibernate générera une colonne dont le nom dérivera de celui de 
la propriété.

```java
package fr.mvc.app.model.entity;

import jakarta.persistence.*;

@Entity // Spécifie que cette classe est une entité
@Table(name = "users") // Nom de la table associée
public class User {

    @Id // Clé primaire
    @GeneratedValue(strategy = GenerationType.IDENTITY) // Auto-incrément
    @Column(name = "id")
    private Long id;

    @Column(name = "username", nullable = false, length = 50)
    private String username;

    @Column(name = "email", nullable = false, length = 100)
    private String email;

    @Column(name = "age")
    private int age;

    // Constructeurs
    public User() {}

    public User(String username, String email, int age) {
        this.username = username;
        this.email = email;
        this.age = age;
    }

    // Getters et setters
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{id=" + id + ", username='" + username + '\'' +
                ", email='" + email + '\'' + ", age=" + age + '}';
    }
}

```

### Cas particulier 

#### Les dates

Pour les dates, si le type est `java.util.Date`ou `java.util.Calendar`, 
il faut définir une annotation supplémentaire `@Temporal(TemporalType.
DATE)`. 

Ce n'est pas utile pour les types `java.time.LocalDate` ou `java.time.LocalDateTime`

```java
@Temporal(TemporalType.DATE)
@Column(name = "birth_date")
private Date birthDate;
```

#### Les grandes quantités de données

Si la propriété peut contenir de grandes quantités de données (fichier binaire ou texte), elle sera marquée avec 
l'annotation `@lob` (Large Object).

```java
@Lob
@Column(name = "profile_picture")
private byte[] profilePicture;
```

```java
@Lob
@Column(name = "content")
private String content;
```

### Les options de l'annotation `@Column`

| Attribut         | Type    | Valeur par défaut | Description                                            |
|------------------|---------|-------------------|--------------------------------------------------------|
| name             | String  | Nom du champ      | Nom de la colonne en base                              |
| nullable         | boolean | true              | Si la colonne accepte les valeurs nulles               |
| unique           | boolean | false             | Si la colonne doit avoir des valeurs uniques           |
| length           | int     | 255               | Longueur maximale pour les types String                |
| precision        | int     | 0                 | Précision pour les types numériques décimaux           |
| scale            | int     | 0                 | Nombre de chiffres après la virgule (décimaux)         |
| insertable       | boolean | true              | Si la colonne est incluse dans les instructions INSERT |
| updatable        | boolean | true              | Si la colonne est incluse dans les instructions UPDATE |
| columnDefinition | String  | Vide              | Spécifie le type SQL directement                       |
| table            | String  | Nom de la table   | Utilisé pour les tables secondaires                    |
| length           | int     | 255               | Longueur de la colonne pour les chaînes                |
| nullable         | boolean | true              | Si la colonne accepte les valeurs NULL                 |

#### Exemples d'annotations avec `@Column`

```java
@Entity
@Table(name = "products")
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "name", nullable = false, length = 100)
    private String name;

    @Column(name = "price", precision = 8, scale = 2)
    private BigDecimal price;

    @Column(name = "sku", unique = true)
    private String sku;

    @Column(name = "created_at", 
            updatable = false, 
            insertable = false, 
            columnDefinition = "TIMESTAMP DEFAULT CURRENT_TIMESTAMP")
    private Timestamp createdAt;

    // Getters et setters
}
```

## Utilisation d'Hibernate

### Persistence

Pour persister une entité, il faut obtenir une instance de la classe `EntityManager`. 


```java

import jakarta.persistence.*;
import fr.mvc.app.model.entity.User;


public class Main {
    
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
        
        // Création d'une entité
        User user = new User(
                "Joe",
                "joe@user.com",
                32
        );
        
        // Persistence d'une entité
        EntityTransaction tx = em.getTransaction();
        
        tx.begin();
        em.persist(user);
        tx.commit();
        
        em.close();
        emf.clos();
    }
}    

```

> Si Hibernate a été configuré en ce sens, la table sera créé automatiquement lors de la persistence. Il n'y a pas de 
migrations à générer et à exécuter.

### Récupération d'une entité

```java

import jakarta.persistence.*;
import fr.mvc.app.model.entity.User;


public class Main {
    
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
        
        // Récupération d'une entité User dont l'id est 1
        User user = em.find(User.class, 1);
        
        if(user != null){
            System.out.prinln(user);
        }

        em.close();
        emf.clos();
    }
}    

```

### Récupération d'une liste d'entités

```java

import jakarta.persistence.*;
import fr.mvc.app.model.entity.User;


public class Main {
    
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
        
        // Récupération d'une entité User dont l'id est 1
        List<User> users = em.createQuery("SELECT u FROM User u", User.class).getResultList();
        for (User user : users) {
            System.out.println(user);
        }

        em.close();
        emf.clos();
    }
}    

```

### Suppression

```java

import jakarta.persistence.*;
import fr.mvc.app.model.entity.User;


public class Main {
    
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

        EntityTransaction tx = em.getTransaction();
        tx.begin();
        User user = em.find(User.class, 1);
        if (user != null) {
            em.remove(user);
            tx.commit();
            System.out.println("Utilisateur supprimé: " + user);
        } else {
            tx.rollback();
            System.out.println("Utilisateur introuvable avec l'ID: 1");
        }
        

        em.close();
        emf.clos();
    }
}    

```

### Mise à jour

Pour la mise à jour, si l'entité est gérée par l'ORM, il n'y a rien à faire. Cette opération est automatiquement 
effectuée lors du commit.

```java

import jakarta.persistence.*;
import fr.mvc.app.model.entity.User;


public class Main {
    
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

        EntityTransaction tx = em.getTransaction();
        tx.begin();
        User user = em.find(User.class, 1);
        if (user != null) {
            user.setUserName("Jane");
            user.setEmail("Jane@user.com");
            user.setAge(37);
            
            // Mise à jour automatique lors du commit
            tx.commit();
            System.out.println("Utilisateur mis à jour: " + user);
        } else {
            tx.rollback();
            System.out.println("Utilisateur introuvable avec l'ID: 1");
        }
        

        em.close();
        emf.clos();
    }
}  

```

#### Entité non gérée

Dans le cas où l'entité ne provient pas de l'ORM, il faudra utiliser la méthode `merge` de `EntityManager`.

```java
import jakarta.persistence.*;
import fr.mvc.app.model.entity.User;


public class Main {
    
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

        EntityTransaction tx = em.getTransaction();
        tx.begin();
        User user = new User();

            user.setId(1);
            user.setUserName("Jane");
            user.setEmail("Jane@user.com");
            user.setAge(37);
            
            // Mise à jour
            em.merge(user);
            tx.commit();
            System.out.println("Utilisateur mis à jour: " + user);
            
            em.close();
            emf.clos();
    }
}
```

## Exercice

Créer une classe UserDAO qui réalise les opérations du CRUD.

### Correction UserDAO {collapsible="true"}

```java
package fr.mvc.app.model.dao;

import fr.mvc.app.model.entity.User;
import javax.persistence.EntityManager;
import javax.persistence.EntityTransaction;
import javax.persistence.Persistence;
import java.util.List;

/**
 * DAO pour l'entité User
 */
public class UserDAO {

    private EntityManager entityManager;

    public UserDAO(String pu) {
        entityManager = Persistence.createEntityManagerFactory(pu)
                                   .createEntityManager();
    }

    public void save(User user) {
        EntityTransaction tx = entityManager.getTransaction();
        try {
            tx.begin();
            entityManager.persist(user);
            tx.commit();
        } catch (Exception e) {
            if (tx.isActive()) tx.rollback();
            e.printStackTrace();
        }
    }

    public User findById(Long id) {
        return entityManager.find(User.class, id);
    }

    public List<User> findAll() {
        return entityManager.createQuery("SELECT u FROM User u", User.class)
                            .getResultList();
    }

    public void update(User user) {
        EntityTransaction tx = entityManager.getTransaction();
        try {
            tx.begin();
            entityManager.merge(user);
            tx.commit();
        } catch (Exception e) {
            if (tx.isActive()) tx.rollback();
            e.printStackTrace();
        }
    }

    public void deleteById(int id) {
        EntityTransaction tx = entityManager.getTransaction();
        try {
            User user = this.findById(id);
            if(user != null){
                tx.begin();
                entityManager.remove(user);
                tx.commit();
            }
        } catch (Exception e) {
            if (tx.isActive()) tx.rollback();
            e.printStackTrace();
        }
    }

    public void close() {
        if (entityManager != null) {
            entityManager.close();
        }
    }
}
```
