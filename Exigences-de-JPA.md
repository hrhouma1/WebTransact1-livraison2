D'accord, je comprends que vous voulez que les liens fonctionnent correctement. Je vais créer des ancres cachées avec des balises HTML pour s'assurer que tout fonctionne correctement.

Voici le document avec les ancres cachées correctement configurées :

---

# 🎯 **Cap sur JPA : La Gestion de Données en Mode Automatique !** 🚀

## **Un voyage pas à pas vers la gestion des données avec JPA et Spring Boot** 📚

---

## 🗂️ **Table des Matières** :

1. [Partie 1 : Exigences Générales JPA](#partie-1)
2. [Partie 2 : Projet de Gestion des Clients et Cartes](#partie-2)
3. [Structure du Projet](#structure-du-projet)
4. [Configuration de la Base de Données](#configuration-de-la-base-de-donnees)
5. [Modèles et Relations JPA](#modeles-et-relations-jpa)
6. [Repositories](#repositories)
7. [Services et Logique Métier](#services-et-logique-metier)
8. [Contrôleurs](#controleurs)
9. [Contraintes d'Intégrité](#contraintes-dintegrite)
10. [Tests et Validation](#tests-et-validation)

---

<!-- Ancre cachée pour Partie 1 -->
<a id="partie-1"></a>
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

---

<!-- Ancre cachée pour Partie 2 -->
<a id="partie-2"></a>
## 🚀 **Partie 2 : Projet de Gestion des Clients et Cartes** 💳

Ce projet est une application Spring Boot gérant **clients** et **cartes bancaires** avec **PostgreSQL** comme base de données.

---

<!-- Ancre cachée pour Structure du Projet -->
<a id="structure-du-projet"></a>
## 🗂️ **Structure du Projet** :

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

---

<!-- Ancre cachée pour Configuration de la Base de Données -->
<a id="configuration-de-la-base-de-donnees"></a>
## ⚙️ **Configuration de la Base de Données** :

Ajoutez les propriétés dans `application.properties` comme vu dans **Partie 1**.

---

<!-- Ancre cachée pour Modèles et Relations JPA -->
<a id="modeles-et-relations-jpa"></a>
## 🏗️ **Modèles et Relations JPA** :

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

---

<!-- Ancre cachée pour Repositories -->
<a id="repositories"></a>
## 📁 **Repositories** :

```java
public interface CustomerRepository extends JpaRepository<Customer, Long> {
    List<Customer> findByNameContaining(String name);
}

public interface CardRepository extends JpaRepository<Card, Long> {
    List<Card> findByCustomerId(Long customerId);
    Optional<Card> findByCardNumber(String cardNumber);
}
```

---

<!-- Ancre cachée pour Services et Logique Métier -->
<a id="services-et-logique-metier"></a>
## 🛠️ **Services et Logique Métier** :

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

---

<!-- Ancre cachée pour Contrôleurs -->
<a id="controleurs"></a>
## 🌐 **Contrôleurs** :

```java
@RestController
@RequestMapping("/api/v1/customers")
public class CustomerController {
    private final CustomerService customerService;

    public CustomerController(CustomerService customerService) {
        this.customerService = customerService;
    }

    @PostMapping
    public ResponseEntity<Customer> createCustomer(@RequestBody Customer customer) {
        return ResponseEntity.ok(customerService.createCustomer(customer));
    }
}
```

---

<!-- Ancre cachée pour Contraintes d'Intégrité -->
<a id="contraintes-dintegrite"></a>
## 🛡️ **Contraintes d'Intégrité** :

1. **Pas de suppression de client** avec des cartes actives 🚫💳.
2. **Chaque carte** doit être associée à un client 🔗.
3. **Le numéro de carte** doit être unique 🔢.

---

<!-- Ancre cachée pour Tests et Validation -->
<a id="tests-et-validation"></a>
## 🧪 **Tests et Validation** :

Créez des tests unitaires pour vérifier les services et les contrôleurs :

```java
@SpringBootTest
class CustomerServiceTest {
    @Autowired
    private CustomerService customerService;

    @Test
    void deleteCustomerWithCardsThrowsException() {
        // Logique de test
    }
}
```

---

Avec ces ancres cachées, vos liens dans la table des matières devraient fonctionner correctement. Les ancres sont maintenant invisibles mais permettent de naviguer facilement dans le document.
