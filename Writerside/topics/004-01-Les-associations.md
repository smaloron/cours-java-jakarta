# Les associations

En JPA (et donc avec Hibernate qui est une implémentation de JPA), les associations entre les entités permettent de
modéliser les relations entre les objets de manière orientée objet, tout en reflétant les relations entre les tables
dans la base de données.
Ces associations sont définies grâce à des annotations au sein des entités.

## Les types d'associations

### OneToOne

La plus simple des associations qui fait correspondre une instance avec une autre instance.

```java

@Entity
public class Person {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;


    private String name;

    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "address_id")
    private Address address;

    // getters et setters
}
```

```java

@Entity
public class Address {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String street;

    // getters et setters
}
```

- La clé étrangère est dans l'entité "Person".

- L'attribut cascade permet de propager les opérations (comme persist, merge, etc.).

- L'annotation `@JoinColumn` spécifie le nom de la clef étrangère qui sera générée. Si cette annotation est omise,
  Hibernate générera un nom de colonne en utilisant le nom de la propriété de l'entité propriétaire et celui de la
  clef primaire de l'entité associée. Donc dans le cas de l'entité `Person` l'annotation est facultative. Toutefois,
  elle peut rendre le code plus explicite.

### ManyToOne unidrectionelle

Soit deux entités : `Order` (commande) et `Customer` (client). 
Une commande est associée à un client, mais le client n'a pas besoin de connaître ses commandes. 
Cela signifie qu'il n'y a pas de relation inverse dans l'entité Customer.

```java
import jakarta.persistence.*;

@Entity
public class Customer {
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;
  
  private String name;

  // Getters et setters
}
```

```java
import jakarta.persistence.*;

@Entity
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String product;

    @ManyToOne()
    private Customer customer;

    // Getters et setters
}
```

### ManyToOne bidirectionnelle

Pour obtenir la liste des commandes liées depuis une entité Customer, 
il faut inverser l'association `ManyToOne` avec une association `OneToMany` dans l'entité cible (ou subordonnée, 
certains emploient même le terme d'esclave).

L'attribut `mappedBy` fait référence à la propriété dans l'entité initiatrice (ici Order) 
qui porte l'association inversée par `OneToMany`.


```java
import jakarta.persistence.*;
import java.util.List;

@Entity
public class Customer {
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;

  private String name;

  @OneToMany(mappedBy = "customer")
  private List<Order> orders;

  // Getters et setters
  
}
```

### ManyToMany
Soit deux entités : Student (étudiant) et Course (cours). 
Dans ce cas, un étudiant peut suivre plusieurs cours, 
et un cours peut être suivi par plusieurs étudiants.

**Entité Student**

- Elle contient une collection des cours (Course) associés.
- L'annotation `@JoinTable` est facultative, mais elle permet de rendre l'association plus explicite.
- La cascade indique les opérations (ici la création et la mise à jour) qui seront réalisée automatiquement sur 
  l'entité liée lors de la persistence de Student.

```java
import jakarta.persistence.*;
import java.util.HashSet;
import java.util.Set;

@Entity
@Table(name = "students")
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
    @JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private Set<Course> courses = new HashSet<>();

    // Getters et setters
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Set<Course> getCourses() {
        return courses;
    }

    public void setCourses(Set<Course> courses) {
        this.courses = courses;
    }

    // Méthodes utilitaires pour synchroniser les deux côtés
    public void addCourse(Course course) {
        this.courses.add(course);
        course.getStudents().add(this);
    }

    public void removeCourse(Course course) {
        this.courses.remove(course);
        course.getStudents().remove(this);
    }
}
```

**L'entité Course**

L'entité Course contient une collection d'étudiants associés. 
Elle utilise l'attribut mappedBy pour indiquer que la relation est définie du côté de l'entité Student.

```Java
import jakarta.persistence.*;
import java.util.HashSet;
import java.util.Set;

@Entity
@Table(name = "courses")
public class Course {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String title;

    @ManyToMany(
            mappedBy = "courses", 
            cascade = {CascadeType.PERSIST, CascadeType.MERGE}
    )
    private Set<Student> students = new HashSet<>();

    // Getters et setters
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public Set<Student> getStudents() {
        return students;
    }

    public void setStudents(Set<Student> students) {
        this.students = students;
    }
}
```

## Les annotations en détails

### @Column

#### Attribut `name`
Spécifie le nom de la colonne dans la base de données.
Si cet attribut n'est pas défini, le nom de la colonne sera dérivé du nom du champ ou de la propriété Java.

**Exemple :**

```java
@Column(name = "customer_name")
private String name;
```

#### Attribut `nullable`
Indique si la colonne peut contenir des valeurs nulles.
Par défaut, la valeur est true, ce qui signifie que les valeurs nulles sont autorisées.

**Exemple :**

```java
@Column(nullable = false)
private String email;
```

#### Attribut unique
Spécifie si les valeurs dans cette colonne doivent être uniques.

Cela ajoute une contrainte d'unicité à la colonne dans le schéma de base de données.

**Exemple :**

```java
@Column(unique = true)
private String username;
```

#### Attribut `length`
Définit la taille maximale pour les colonnes de type String.

Par défaut, la valeur est 255.

**Exemple :**

```java
@Column(length = 100)
private String description;
```

#### Attributs `precision` et `scale`
Utilisés pour les colonnes de type décimal (BigDecimal) :

- precision : Nombre total de chiffres (avant et après la virgule).

- scale : Nombre de chiffres après la virgule.

**Exemple :**

```java
@Column(precision = 10, scale = 2)
private BigDecimal price;
```

#### Attribut `insertable`
Indique si cette colonne doit être incluse dans les instructions SQL INSERT.

Par défaut, la valeur est true.

**Exemple :**

```java
@Column(insertable = false)
private String createdBy;
```

#### Attribut `updatable`
Indique si cette colonne doit être incluse dans les instructions SQL UPDATE.

Par défaut, la valeur est true.

**Exemple :**

```java
@Column(updatable = false)
private LocalDate createdDate;
```

#### Attribut `columnDefinition`
Permet de spécifier directement le type SQL ou toute autre définition spécifique à la base de données 
pour cette colonne.

Utile pour des types spécifiques ou des contraintes non standard.

**Exemple :**

```java
@Column(columnDefinition = "TEXT NOT NULL")
private String longDescription;
```

#### Attribut `table`
Permet de spécifier le nom de la table dans laquelle se trouve cette colonne 
(utile si l'entité est mappée sur plusieurs tables).

Par défaut, la table principale est utilisée.

**Exemple :**

```java
@Column(table = "customer_details")
private String address;
```

### Les attributs des associations (@OneToOne, @ManyToOne, @OneToMany et @ManyToMany)

#### Attribut `mappedBy`

Utilisé pour spécifier le côté non propriétaire (inverse) dans une relation bidirectionnelle.

Il indique le champ ou la propriété qui mappe la relation dans l'entité propriétaire.

**Exemple :**

```java
@OneToOne(mappedBy = "customerRecord")
private Customer customer;
```

Ici, customerRecord est le champ dans l'entité propriétaire.

#### Attribut `cascade`
Définit les opérations à propager (cascader) de l'entité source vers l'entité cible.

Valeurs possibles :

- CascadeType.PERSIST

- CascadeType.MERGE

- CascadeType.REMOVE

- CascadeType.REFRESH

- CascadeType.DETACH

- CascadeType.ALL (toutes les opérations).

**Exemple :**

```java
@OneToOne(cascade = CascadeType.ALL)
private Address address;
```
<br/>

L'attribut cascade admet également un tableau de valeurs

```java
@OneToOne(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
private Address address;
```

#### Attribut `fetch`
Spécifie si la relation doit être chargée de manière paresseuse (FetchType.LAZY) ou immédiate (FetchType.EAGER).

Par défaut : FetchType.EAGER.

**Exemple :**

```java
@OneToOne(fetch = FetchType.LAZY)
private Profile profile;
```

#### Attribut `optional`

Indique si la relation est facultative ou non.

Si défini sur false, une exception sera levée si la relation n'est pas définie.

Par défaut : true.

**Exemple :**

```java
@OneToOne(optional = false)
private Passport passport;
```

#### Attribut `orphanRemoval`
Permet de supprimer automatiquement l'entité cible si elle est dissociée de l'entité source.

Cela équivaut à utiliser CascadeType.REMOVE, mais uniquement pour les entités orphelines.

**Exemple :**

```java
@OneToOne(orphanRemoval = true)
private UserProfile userProfile;
```

#### Attribut `targetEntity`
Spécifie explicitement la classe cible de la relation.

Utile lorsque le type ne peut pas être déterminé automatiquement, comme avec des types génériques.

Par défaut : déduit du type du champ ou de la propriété.

**Exemple :**

```java
@OneToOne(targetEntity = Address.class)
private Object address;
```

### L'annotation @JoinColumn

@JoinColumn est utilisée pour personnaliser et configurer la colonne de clé étrangère 
dans une relation entre deux entités. 

Cette annotation possède les mêmes attributs que l'annotation @Column et en ajoute une pour gérer les clefs 
étrangères particulières.


#### Attribut `referencedColumnName`
Spécifie le nom de la colonne dans l'entité référencée (par défaut, c'est la clé primaire).

Utile lorsque la clé étrangère ne pointe pas vers la colonne par défaut (la clé primaire).

**Exemple :**

```java
@JoinColumn(name = "address_zip", referencedColumnName = "zipcode")
private Address address;
```



