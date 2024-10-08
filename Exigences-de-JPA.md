# 🎯 **Cap sur JPA : La Gestion de Données en Mode Automatique !** 🚀

## **Un voyage pas à pas vers la gestion des données avec JPA et Spring Boot** 📚

---

## 🗂️ **Table des Matières** :

1. [Partie 1 : Exigences Générales JPA](#partie-1-exigences-generales-jpa)
2. [Partie 2 : Projet de Gestion des Clients et Cartes](#partie-2-projet-de-gestion-des-clients-et-cartes)
3. [Structure du Projet](#structure-du-projet)
4. [Configuration de la Base de Données](#configuration-de-la-base-de-donnees)
5. [Modèles et Relations JPA](#modeles-et-relations-jpa)
6. [Repositories](#repositories)
7. [Services et Logique Métier](#services-et-logique-metier)
8. [Contrôleurs](#controleurs)
9. [Contraintes d'Intégrité](#contraintes-dintegrite)
10. [Tests et Validation](#tests-et-validation)

---

## 🏁 **Partie 1 : Exigences Générales JPA** 🎓

| 📂 **Catégorie** | ⚙️ **Exigence** | 📝 **Description** |
|-----------|----------|-------------|
| **pom.xml** | 📦 Dépendance JPA | ➡️ Ajouter `spring-boot-starter-data-jpa` |
| **pom.xml** | 🐘 Dépendance PostgreSQL | ➡️ Ajouter le driver PostgreSQL |
| **application.properties** | 🏗️ Configuration de la base de données | ➡️ Ajouter les propriétés de connexion à PostgreSQL |
| **application.properties** | 🏗️ Configuration JPA | ➡️ Configurer le comportement de JPA/Hibernate |
| **Modèles** | 📍 Annotations JPA | ➡️ Ajouter les annotations nécessaires aux classes de modèle |
| **Modèles** | 🆔 Identifiant | ➡️ S'assurer que chaque entité a un identifiant unique |
| **Modèles** | 🛠️ Constructeurs | ➡️ Avoir un constructeur par défaut (sans argument) |
| **Repository** | 📝 Interface Repository | ➡️ Créer des interfaces repository étendant JpaRepository |

Détaillons chaque point :

1. **Ajoutez dans pom.xml** 📄 :
   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-jpa</artifactId>
   </dependency>
   <dependency>
       <groupId>org.postgresql</groupId>
       <artifactId>postgresql</artifactId>
       <scope>runtime</scope>
   </dependency>
   ```
   💡 Cela permet à votre projet d’utiliser **JPA** et de se connecter à **PostgreSQL** 🐘.

2. **Ajoutez dans application.properties** :
   ```properties
   spring.datasource.url=jdbc:postgresql://localhost:5432/your_database_name
   spring.datasource.username=your_username
   spring.datasource.password=your_password
   spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
   spring.jpa.hibernate.ddl-auto=update
   ```
   🔑 **Important** : Modifiez les valeurs pour correspondre à votre configuration locale !

3. **Ajoutez les annotations JPA dans vos classes de modèle** :
   ```java
   @Entity
   @Table(name = "customers")
   public class Customer {
       @Id
       @GeneratedValue(strategy = GenerationType.IDENTITY)
       private Long id;
       
       @Column(name = "name")
       private String name;
       
       // Constructeur sans argument
       public Customer() {}
       
       // Getters et setters
   }
   ```

4. **Créez une interface repository** :
   ```java
   public interface CustomerRepository extends JpaRepository<Customer, Long> {
   }
   ```

5. **Vérifiez que vos modèles respectent les exigences JPA** :
   - 🎯 **Chaque classe** doit avoir l'annotation `@Entity`
   - 🆔 **Chaque entité** doit avoir un identifiant unique annoté avec `@Id`
   - 🧑‍🤝‍🧑 Les relations entre entités doivent être annotées correctement (`@OneToMany`, `@ManyToOne`, etc.)
   - 🔧 **Constructeur par défaut** obligatoire.

6. **Si vous utilisez Lombok** :
   ```java
   @Entity
   @Table(name = "customers")
   @Data
   @NoArgsConstructor
   @AllArgsConstructor
   public class Customer {
       @Id
       @GeneratedValue(strategy = GenerationType.IDENTITY)
       private Long id;
       
       @Column(name = "name")
       private String name;
   }
   ```

7. **PostgreSQL installé et en cours d'exécution** : ✅ **Assurez-vous que PostgreSQL est prêt** et que la base de données est configurée.

---

## 🚀 **Partie 2 : Projet de Gestion des Clients et Cartes** 💳

Ce projet est une application Spring Boot gérant **clients** et **cartes bancaires** avec **PostgreSQL** comme base de données.

### 🗂️ **Structure du Projet** :

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

### ⚙️ **Configuration de la Base de Données** :

Ajoutez les propriétés dans `application.properties` comme vu dans **Partie 1**.

---

### 🏗️ **Modèles et Relations JPA** :

```java
@Entity
@Table(name = "customers")
public class Customer {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String name;
    
    @OneToMany(mappedBy = "customer", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Card> cards = new ArrayList<>();

    // Getters, setters, etc.
}

@Entity
@Table(name = "cards")
public class Card {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true)
    private String cardNumber;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "customer_id", nullable = false)
    private Customer customer;

    // Autres champs, getters, setters, etc.
}
```
📌 **Note** : Les relations sont définies entre **Customer** et **Card** avec `@OneToMany` et `@ManyToOne`.

---

### 📁 **Repositories** :

```java
public interface CustomerRepository extends JpaRepository<Customer, Long> {
    List<Customer> findByNameContaining(String name);
}

public interface CardRepository extends JpaRepository<Card, Long> {
    List<Card> findByCustomerId(Long customerId);
    Optional<Card> findByCardNumber(String cardNumber);
}
```

🎯 **JpaRepository** vous donne accès à des méthodes CRUD et vous permet d'ajouter des méthodes personnalisées.

---

### 🛠️ **Services et Logique Métier** :

```java
@Service
@Transactional
public class CustomerService {
    private final CustomerRepository customerRepository;

    public Customer createCustomer(Customer customer) {
        return customerRepository.save(customer);
    }

    public void deleteCustomer(Long id) {
        Customer customer = customerRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Customer not found"));
        customerRepository.delete(customer);
    }
}
```
🧑‍💻 **Les services** contiennent la logique métier et gèrent les transactions.

---

### 🌐 **Contrôleurs** :

```java
@RestController
@RequestMapping("/api/v1/customers")
public class CustomerController {
    @PostMapping
    public ResponseEntity<Customer> createCustomer(@RequestBody Customer customer) {
        return ResponseEntity.ok(customerService.createCustomer(customer));
    }
}
```
📥 **Les contrôleurs** gèrent les requêtes HTTP.

---

### 🛡️ **Contraintes d'Intégrité** :

1. **Pas de suppression de client** avec des cartes actives 🚫💳.
2. **Chaque carte** doit être associée à un client 🔗.
3. **Le numéro de carte** doit être unique 🔢.

---

### 🧪 **Tests et Validation** :

Créez des tests unitaires pour vérifier les services et les contrôleurs :

```java
@SpringBootTest
class CustomerServiceTest {
    @Autowired
    private CustomerService customerService;

    @Test
    void deleteCustomerWithCardsThrowsException() {
        // Test logic
    }
}
```
