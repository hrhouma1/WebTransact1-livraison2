--------------------------------------------
# **Projet de Gestion des Clients et Cartes Bancaires** 🚀
--------------------------------------------

- Ce document vous guide de façon **exhaustive** à travers toutes les étapes nécessaires pour configurer, développer et tester une application Spring Boot avec JPA et PostgreSQL. 

## Annexe 1 - **Résumé des Commandes Maven** 🛠️  
Un tableau récapitulatif des principales commandes Maven, incluant la combinaison de commandes pour une exécution efficace du projet.

## Annexe 2 - **Détails des Types et Relations JPA** 📚  
Explication détaillée des types de données utilisés dans JPA, comme le rôle de `Long` en tant que type de clé primaire dans les entités, et les implications sur la gestion des relations entre entités.

## Annexe 3 - **Guide des Relations JPA : `JoinColumn` et `mappedBy`** 🔄  
Une analyse approfondie de l’utilisation des annotations `@JoinColumn` et `mappedBy` dans les relations entre entités, avec exemples concrets pour bien comprendre la structure des clés étrangères.


--------------------------------------------
# **Table des Matières** 🗂️
--------------------------------------------


1. [Aperçu du Projet](#aperçu-du-projet)
2. [Structure du Projet](#structure-du-projet)
3. [Configuration](#configuration)
4. [Modèles et Relations](#modèles-et-relations)
5. [Repositories](#repositories)
6. [Services](#services)
7. [Contrôleurs](#contrôleurs)
8. [Contraintes d'Intégrité](#contraintes-dintégrité)
9. [Fonctionnalités Principales](#fonctionnalités-principales)
10. [Exécution du Projet](#exécution-du-projet)
11. [Tests](#tests)

---

<!-- Ancre cachée pour Table des matières -->
<a id="table-des-matieres"></a>
# 🌟 **Aperçu du Projet**

Dans ce projet, vous allez développer une application Spring Boot pour gérer des **clients** et leurs **cartes bancaires**, en utilisant **PostgreSQL** comme base de données. Cette application met en œuvre l'architecture en couches, avec **JPA** pour la gestion de la persistance des données. Vous apprendrez à manipuler des relations entre entités comme **OneToMany** et **ManyToOne**, tout en garantissant une sérialisation JSON correcte avec **Jackson**.

[Retour en haut](#table-des-matieres)

---

<!-- Ancre cachée pour Structure du projet -->
<a id="structure-du-projet"></a>
# 🏗 **Structure du Projet**

Voici comment les composants principaux de votre projet sont structurés. Chaque partie a un rôle spécifique : 
- **Controller** : Reçoit et traite les requêtes HTTP.
- **Service** : Contient la logique métier.
- **Repository** : Interagit avec la base de données.
- **Model** : Définit les entités et leurs relations.

```
src/main/java/com/example/demo/
├── config/
│   └── OpenAPIConfig.java
├── controller/
│   ├── CardController.java
│   └── CustomerController.java
├── model/
│   ├── Card.java
│   └── Customer.java
├── repository/
│   ├── CardRepository.java
│   └── CustomerRepository.java
├── service/
│   ├── CardService.java
│   └── CustomerService.java
└── exception/
    └── ResourceNotFoundException.java
```

[Retour en haut](#table-des-matieres)

---

<!-- Ancre cachée pour Configuration -->
<a id="configuration"></a>
# ⚙ **Configuration**

Nous allons commencer par configurer votre projet. Assurez-vous que les dépendances nécessaires sont bien incluses dans **pom.xml** et que les propriétés de connexion à la base de données sont correctement définies dans **application.properties**.

### **1. pom.xml**

Ajoutez les dépendances nécessaires pour Spring Boot, JPA, et PostgreSQL :

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

### **2. application.properties**

Configurez la connexion à la base de données et le comportement de JPA dans ce fichier :

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/your_database_name
spring.datasource.username=your_username
spring.datasource.password=your_password
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.hibernate.ddl-auto=update
```

[Retour en haut](#table-des-matieres)

---

<!-- Ancre cachée pour Modèles et Relations -->
<a id="modèles-et-relations"></a>
# 📊 **Modèles et Relations**

Les modèles représentent vos entités. Dans ce projet, vous avez deux entités principales : **Customer** et **Card**. Un **Customer** peut posséder plusieurs **Cards** (relation **OneToMany**), tandis qu'une **Card** est liée à un seul **Customer** (relation **ManyToOne**).

### **Customer.java**

Voici le modèle pour la classe **Customer**. Il possède une liste de cartes et chaque carte appartient à un client. Cette relation est gérée avec les annotations JPA **@OneToMany** et **@ManyToOne** pour établir le lien entre les entités.

```java
@Entity
@Table(name = "customers")
public class Customer {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long customerId;

    @Column(name = "last_name", nullable = false)
    private String customerLastName;

    @Column(name = "first_name", nullable = false)
    private String customerFirstName;

    @OneToMany(mappedBy = "customer", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Card> cards = new ArrayList<>();
}
```

### **Card.java**

Le modèle **Card** contient un lien vers le **Customer** correspondant. L'annotation **@ManyToOne** indique que chaque carte est associée à un client.

```java
@Entity
@Table(name = "cards")
public class Card {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long cardId;

    @Column(nullable = false, unique = true)
    private String cardNumber;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "customer_id", nullable = false)
    private Customer customer;
}
```

[Retour en haut](#table-des-matieres)

---

<!-- Ancre cachée pour Repositories -->
<a id="repositories"></a>
# 📚 **Repositories**

Les **repositories** sont utilisés pour interagir avec la base de données. Les interfaces **CustomerRepository** et **CardRepository** étendent **JpaRepository**, fournissant des méthodes de base pour manipuler les entités.

### **CustomerRepository.java**

```java
@Repository
public interface CustomerRepository extends JpaRepository<Customer, Long> {
    List<Customer> findByCustomerLastNameContainingOrCustomerFirstNameContaining(String lastName, String firstName);
}
```

### **CardRepository.java**

```java
@Repository
public interface CardRepository extends JpaRepository<Card, Long> {
    List<Card> findByCustomerId(Long customerId);
    Optional<Card> findByCardNumber(String cardNumber);
}
```

[Retour en haut](#table-des-matieres)

---

<!-- Ancre cachée pour Services -->
<a id="services"></a>
# 🛠 **Services**

Les **services** encapsulent la logique métier de l'application. Ils gèrent les opérations sur les entités et délèguent les opérations de persistance aux repositories.

### **CustomerService.java**

Le service **CustomerService** gère la création, la suppression, et la récupération des clients.

```java
@Service
@Transactional
public class CustomerService {
    private final CustomerRepository customerRepository;
    private final CardRepository cardRepository;

    public Customer createCustomer(Customer customer) {
        return customerRepository.save(customer);
    }

    public List<Customer> getAllCustomers() {
        return customerRepository.findAll();
    }

    public void deleteCustomer(Long id) {
        Customer customer = customerRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Customer not found"));
        customerRepository.delete(customer);
    }
}
```

### **CardService.java**

Le service **CardService** gère la création de cartes et leur association avec un client.

```java
@Service
@Transactional
public class CardService {
    private final CardRepository cardRepository;
    private final CustomerRepository customerRepository;

    public Card createCard(Card card, Long customerId) {
        Customer customer = customerRepository.findById(customerId)
            .orElseThrow(() -> new ResourceNotFoundException("Customer not found"));
        card.setCustomer(customer);
        return cardRepository.save(card);
    }

    public List<Card> getAllCards() {
        return cardRepository.findAll();
    }
}
```

[Retour en haut](#table-des-matieres)

---

<!-- Ancre cachée pour Contrôleurs -->
<a id="contrôleurs"></a>
# 🎮 **Contrôleurs**

Les **contrôleurs** exposent des API REST pour interagir avec les services. Ils reçoivent les requêtes HTTP, appellent les services correspondants et renvoient les réponses.

### **CustomerController.java**

```java
@RestController
@RequestMapping("/api/v1/customers")
public class CustomerController {
    private final CustomerService customerService;

    public CustomerController(CustomerService customerService) {
        this.customerService = customerService;
    }

    @PostMapping
    public ResponseEntity<Customer> addCustomer(@RequestBody Customer customer) {
        return ResponseEntity.ok(customerService.createCustomer(customer));
    }

    @GetMapping
    public ResponseEntity<List<Customer>> getAllCustomers() {
        return ResponseEntity.ok(customerService.getAllCustomers());
    }
}
```

### **CardController.java**

```java
@RestController
@RequestMapping("/api/v1/cards")
public class CardController {
    private final CardService cardService;

    public CardController(CardService cardService) {
        this.cardService = cardService;
    }



    @PostMapping
    public ResponseEntity<Card> addCard(@RequestBody Card card, @RequestParam Long customerId) {
        return ResponseEntity.ok(cardService.createCard(card, customerId));
    }

    @GetMapping
    public ResponseEntity<List<Card>> getAllCards() {
        return ResponseEntity.ok(cardService.getAllCards());
    }
}
```

[Retour en haut](#table-des-matieres)

---

<!-- Ancre cachée pour Contraintes d'Intégrité -->
<a id="contraintes-dintégrité"></a>
# 🔒 **Contraintes d'Intégrité**

1. **Un client ne peut pas être supprimé s'il a des cartes actives**.
2. **Une carte doit toujours être associée à un client existant**.
3. **Le numéro de carte doit être unique**.

[Retour en haut](#table-des-matieres)

---

<!-- Ancre cachée pour Fonctionnalités Principales -->
<a id="fonctionnalités-principales"></a>
# 🚀 **Fonctionnalités Principales**

- **CRUD complet** pour les clients et les cartes.
- **Recherche de clients** par nom ou prénom.
- **Gestion des relations** entre clients et cartes.
- **Validation des données** et gestion des erreurs.
- **Documentation API avec Swagger**.

[Retour en haut](#table-des-matieres)

---

<!-- Ancre cachée pour Exécution du Projet -->
<a id="exécution-du-projet"></a>
# 🏃‍♂️ **Exécution du Projet**

1. Assurez-vous que PostgreSQL est en cours d'exécution.
2. Configurez la base de données dans `application.properties`.
3. Exécutez : `./mvnw spring-boot:run`.
4. Accédez à l'API : `http://localhost:8080`.
5. Accédez à la documentation Swagger : `http://localhost:8080/swagger-ui.html`.

[Retour en haut](#table-des-matieres)

---

<!-- Ancre cachée pour Tests -->
<a id="tests"></a>
# 🧪 **Tests**

Les tests permettent de vérifier le bon fonctionnement de votre application. Un exemple simple est de tester que la suppression d'un client avec des cartes actives génère bien une exception.

### **CustomerServiceTest.java**

```java
@SpringBootTest
class CustomerServiceTest {

    @Autowired
    private CustomerService customerService;

    @Test
    void deleteCustomerWithCardsThrowsException() {
        // Logique de test pour vérifier que la suppression d'un client avec des cartes génère une exception.
    }
}
```

[Retour en haut](#table-des-matieres)

---

# Annexe 1 


Voici une table résumant quelques commandes Maven importantes, y compris la combinaison que tu souhaites utiliser (`mvn clean install -DskipTests spring-boot:run`).

| **Commande**                                         | **Description**                                                                                   |
|------------------------------------------------------|---------------------------------------------------------------------------------------------------|
| `mvn clean`                                          | Nettoie le répertoire `target` des fichiers générés dans les builds précédents.                    |
| `mvn compile`                                        | Compile le code source de ton projet.                                                             |
| `mvn test`                                           | Exécute les tests unitaires configurés dans le projet.                                             |
| `mvn package`                                        | Emballe le projet dans un fichier JAR ou WAR.                                                      |
| `mvn install`                                        | Installe le package dans le repository Maven local pour une utilisation par d'autres projets.      |
| `mvn clean install`                                  | Nettoie, compile, teste et emballe l'application, puis installe le package localement.             |
| `mvn clean install -DskipTests`                      | Fait tout le processus d'installation sans exécuter les tests unitaires.                           |
| `mvn spring-boot:run`                                | Démarre l'application Spring Boot directement depuis le code source.                               |
| **`mvn clean install -DskipTests spring-boot:run`**  | Nettoie, compile, emballe l'application sans exécuter les tests, puis la démarre.                  |
| `mvn spring-boot:build-image`                        | Crée une image Docker pour l'application Spring Boot.                                              |
| `mvn deploy`                                         | Déploie le package sur un dépôt Maven distant.                                                     |

En combinant **`mvn clean install -DskipTests spring-boot:run`**, tu effectues le processus d'installation sans les tests puis tu lances l'application Spring Boot.


# Annexe 2


Dans l'interface `JpaRepository<Customer, Long>`, le second paramètre, `Long`, représente le **type de l'identifiant** (ID) de l'entité `Customer`. Voici une explication plus détaillée :

- **`Customer`** : c'est l'entité que vous gérez dans votre base de données. Chaque enregistrement dans la table associée à `Customer` correspondra à une instance de cette classe.
- **`Long`** : cela indique le type de la clé primaire (ID) de l'entité `Customer`.

Dans JPA, chaque entité doit avoir un identifiant unique, et cet identifiant est souvent un type de données comme `Long`, `Integer`, ou même `UUID`. Le type que vous choisissez pour l'ID dépend de la façon dont vous souhaitez gérer les clés primaires dans la base de données.

Dans votre classe `Customer`, l'ID est défini comme `Long` :
```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

Ainsi, dans `JpaRepository<Customer, Long>`, `Long` fait référence au type de cette clé primaire, c'est-à-dire le type de l'attribut `id` dans l'entité `Customer`. Cela permet à **JpaRepository** de savoir que les opérations CRUD se feront sur une entité `Customer` avec un identifiant de type `Long`.

---

### Pour résumer :
- **Le premier paramètre** (`Customer`) représente l'entité que vous gérez.
- **Le second paramètre** (`Long`) est le type de l'ID (la clé primaire) de cette entité.

Si votre ID avait été un autre type (comme `UUID` ou `Integer`), vous auriez mis ce type à la place de `Long`.

# Annexe 3 

Voici une manière de formuler une question par rapport à cela :

**Question :**

Dans une relation entre deux entités `A` et `B` en JPA, comment déterminer quelle entité doit utiliser `@JoinColumn` et quelle entité doit utiliser `mappedBy` ? Expliquez également les conséquences de cette décision sur la structure de la base de données et la gestion des clés étrangères.


En général, pour définir une relation entre entités en JPA ou Hibernate, il n'y a pas de préférence stricte pour l'ordre dans lequel tu mets la relation d'origine (celle qui contient l'annotation `@JoinColumn`) par rapport à celle qui est `mappedBy`. Cependant, l'important est de bien comprendre les rôles respectifs de chaque annotation et comment elles fonctionnent ensemble.

### 1. **`@JoinColumn` :**
   - Elle est placée du côté **propriétaire** de la relation. Cela signifie que c'est cette entité qui possède la clé étrangère et qui est responsable de la gestion de la relation dans la base de données.
   - Par exemple, dans une relation Many-to-One, c'est souvent l'entité qui a une référence vers l'autre qui est propriétaire de la relation.

### 2. **`mappedBy` :**
   - Elle est utilisée du côté **inverse** de la relation, c'est-à-dire celui qui ne possède pas la clé étrangère dans la base de données. Elle indique que la relation est déjà définie ailleurs et qu'il ne faut pas créer une nouvelle colonne pour cette relation.

### Exemple :
Si tu as une relation entre `Employe` et `Departement` (un employé appartient à un département), voici deux façons de la définir :

1. **Avec `@JoinColumn` dans `Employe` (propriétaire)** :
   ```java
   @Entity
   public class Employe {
       @ManyToOne
       @JoinColumn(name = "departement_id") // Employe contient la clé étrangère vers Departement
       private Departement departement;
   }
   ```

2. **Avec `mappedBy` dans `Departement`** :
   ```java
   @Entity
   public class Departement {
       @OneToMany(mappedBy = "departement")
       private List<Employe> employes;
   }
   ```

Dans cet exemple, c'est `Employe` qui contient la clé étrangère vers `Departement`, et `Departement` indique que la relation est déjà définie dans `Employe` via `mappedBy`.

### Choix de l'emplacement
- **Si tu veux contrôler où la clé étrangère est stockée**, utilise `@JoinColumn` du côté qui possède la relation.
- **Si tu veux simplement naviguer dans la relation**, sans créer une nouvelle colonne, utilise `mappedBy`.

Donc, tu peux effectivement définir la relation d'une part ou d'une autre, tant que tu respectes cette logique de propriétaire et de côté inverse.



