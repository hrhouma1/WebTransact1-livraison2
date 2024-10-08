-----------------------------
# Étapes 
-----------------------------

1. Je commence par le **pom.xml**.
2. Je vérifie le fichier **application.properties** :

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/microDemo1
spring.datasource.username=hrgres
spring.datasource.password=hrgres
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.hibernate.ddl-auto=update
```

3. **Retravailler le modèle Card** (voir **Annexe 1**).
4. **Retravailler le modèle Customer** (voir **Annexe 2**).
5. **Retravailler les contrôleurs** (voir **Annexe 3**).

> **Erreur** : Les contrôleurs référencent des services qui n'existent pas encore.

```bash
Changements principaux :
- Utilisation de `CardService` au lieu de manipuler directement une liste de cartes.
- Changement des types `int` en `Long` pour les IDs afin de correspondre au type généré par JPA.
- Suppression de la dépendance directe à `CustomerController`.
- Ajout d'une méthode pour obtenir les cartes d'un client spécifique.
- Utilisation cohérente de `ResponseEntity` pour toutes les réponses.
- Adaptation des méthodes pour utiliser les services JPA au lieu des opérations en mémoire.

Ce contrôleur est maintenant prêt à être utilisé avec JPA et un service correspondant, tout en conservant la documentation Swagger.
```

6. **Ajout du package service** et du package **exception** (voir **Annexe 4**).
7. **Ajout du package repository** et des classes (voir **Annexe 5**).
8. **Corrigez les erreurs dans le contrôleur** ou tout autre problème qui pourrait survenir (voir **Annexe 6** et **READMEplus.md**).

```bash
Points Clés :
- Passage de l'ID du client : Assurez-vous que l'ID du client est passé en tant que paramètre `@RequestParam` lors de l'ajout d'une nouvelle carte.
- Gestion des exceptions : Utilisation de `ResourceNotFoundException` pour gérer les cas où le client ou la carte n'existent pas.
- Utilisation cohérente de `ResponseEntity` pour toutes les réponses HTTP.

Ces modifications devraient résoudre l'erreur et permettre à votre contrôleur et service de fonctionner correctement ensemble.
```

-----------------------------
# **Annexe 1 - Retravailler le modèle Card**
-----------------------------

### Ancienne version :

```java
package com.example.demo.model;

import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class Card {
    private int cardId;
    private int customerId;
    private String cardNumber;
    private String cardType;
    private int totalLimit;
    private int amountUsed;
    private int availableAmount;
    private String createDt;
}
```

### Nouvelle version :

Voici la classe **Card** réécrite pour respecter **Lombok** et **JPA** :

```java
package com.example.demo.model;

import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;
import lombok.Builder;

import java.time.LocalDate;

import jakarta.persistence.*;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
@Entity
@Table(name = "cards")
public class Card {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long cardId;

    @Column(nullable = false, unique = true)
    private String cardNumber;

    @Column(nullable = false)
    private String cardType;

    @Column(nullable = false)
    private int totalLimit;

    @Column(nullable = false)
    private int amountUsed;

    @Column(nullable = false)
    private int availableAmount;

    @Column(nullable = false)
    private LocalDate createDt;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "customer_id", nullable = false)
    private Customer customer;
}
```

Changements et explications :
1. Ajout des annotations **JPA** : `@Entity`, `@Table`, `@Id`, `@GeneratedValue`, `@Column`, `@ManyToOne`, `@JoinColumn`.
2. Changement du type de `cardId` de `int` à `Long` pour correspondre au type généré par la stratégie d'identité.
3. Suppression de `customerId` et ajout d'une relation `@ManyToOne` avec **Customer**.
4. Ajout de contraintes `nullable = false` pour les champs obligatoires.
5. Ajout de `unique = true` pour **cardNumber** pour garantir son unicité.
6. Changement du type de `createDt` de `String` à `LocalDate` pour une meilleure gestion des dates.
7. Ajout de l'annotation **@Builder** de **Lombok** pour permettre la création d'instances avec un pattern builder.

-----------------------------
# **Annexe 2 - Retravailler le modèle Customer**
-----------------------------

### Ancienne version :

```java
package com.example.demo.model;

public class Customer {
    private int customerId;
    private String customerLastName;
    private String customerFirstName;

    public Customer() {}

    public Customer(int customerId, String customerLastName, String customerFirstName) {
        this.customerId = customerId;
        this.customerLastName = customerLastName;
        this.customerFirstName = customerFirstName;
    }

    public int getCustomerId() {
        return customerId;
    }

    public void setCustomerId(int customerId) {
        this.customerId = customerId;
    }

    public String getCustomerLastName() {
        return customerLastName;
    }

    public void setCustomerLastName(String customerLastName) {
        this.customerLastName = customerLastName;
    }

    public String getCustomerFirstName() {
        return customerFirstName;
    }

    public void setCustomerFirstName(String customerFirstName) {
        this.customerFirstName = customerFirstName;
    }
}
```

### Nouvelle version :

Voici la classe **Customer** retravaillée pour respecter les exigences de **JPA** et utiliser **Lombok** pour réduire le code :

```java
package com.example.demo.model;

import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;
import lombok.Builder;

import jakarta.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
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
    @Builder.Default
    private List<Card> cards = new ArrayList<>();

    // Méthode utilitaire pour ajouter une carte
    public void addCard(Card card) {
        cards.add(card);
        card.setCustomer(this);
    }

    // Méthode utilitaire pour supprimer une carte
    public void removeCard(Card card) {
        cards.remove(card);
        card.setCustomer(null);
    }
}
```

Changements et explications :
1. Ajout des annotations **JPA** : `@Entity`, `@Table`, `@Id`, `@GeneratedValue`, `@Column`, `@OneToMany`.
2. Utilisation des annotations **Lombok** : `@Data`, `@NoArgsConstructor`, `@AllArgsConstructor`, `@Builder`.
3. Changement du type de **customerId** de `int` à `Long`.
4. Ajout de la relation `@OneToMany` avec **Card**.
5. Ajout de méthodes utilitaires **addCard** et **removeCard** pour gérer la relation bidirectionnelle avec **Card**.

-----------------------------
# **Annexe 3 - Retravailler les contrôleurs**
-----------------------------

### Nouvelle version de **CardController** et **CustomerController** :

1. **CardController.java** :

```java
package com.example.demo.controller;

import com.example.demo.model.Card;
import com.example.demo.service.CardService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.media.Content;
import io.swagger.v3.oas.annotations.media.Schema;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.tags.Tag;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/v1/cards")
@Tag(name = "Card", description = "API pour la gestion des cartes bancaires")
public class CardController {

    private final CardService cardService;

    @Autowired
    public CardController(CardService cardService) {
        this.cardService = cardService;
    }

    @Operation(summary = "Ajouter une nouvelle carte", description = "Ajoute une nouvelle carte bancaire pour un client existant")
    @ApiResponse(responseCode = "200", description = "Carte ajoutée avec succès", content = @Content(schema = @Schema(implementation = Card.class)))
    @ApiResponse(responseCode = "400", description = "Données invalides ou client inexistant")
    @PostMapping
    public ResponseEntity<Card> addCard(@RequestBody Card card, @RequestParam Long customerId)

 {
        return ResponseEntity.ok(cardService.createCard(card, customerId));
    }

    @Operation(summary = "Obtenir toutes les cartes", description = "Récupère la liste de toutes les cartes bancaires")
    @ApiResponse(responseCode = "200", description = "Liste des cartes récupérée avec succès")
    @GetMapping
    public ResponseEntity<List<Card>> getAllCards() {
        return ResponseEntity.ok(cardService.getAllCards());
    }

    @Operation(summary = "Obtenir une carte par ID", description = "Récupère les détails d'une carte bancaire spécifique")
    @ApiResponse(responseCode = "200", description = "Carte trouvée", content = @Content(schema = @Schema(implementation = Card.class)))
    @ApiResponse(responseCode = "404", description = "Carte non trouvée")
    @GetMapping("/{id}")
    public ResponseEntity<Card> getCardById(@Parameter(description = "ID de la carte") @PathVariable Long id) {
        return cardService.getCardById(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    @Operation(summary = "Mettre à jour une carte", description = "Met à jour les détails d'une carte bancaire existante")
    @ApiResponse(responseCode = "200", description = "Carte mise à jour avec succès", content = @Content(schema = @Schema(implementation = Card.class)))
    @ApiResponse(responseCode = "404", description = "Carte non trouvée")
    @PutMapping("/{id}")
    public ResponseEntity<Card> updateCard(@Parameter(description = "ID de la carte") @PathVariable Long id, @RequestBody Card updatedCard) {
        return ResponseEntity.ok(cardService.updateCard(id, updatedCard));
    }

    @Operation(summary = "Supprimer une carte", description = "Supprime une carte bancaire spécifique")
    @ApiResponse(responseCode = "200", description = "Carte supprimée avec succès")
    @ApiResponse(responseCode = "404", description = "Carte non trouvée")
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteCard(@Parameter(description = "ID de la carte") @PathVariable Long id) {
        cardService.deleteCard(id);
        return ResponseEntity.ok().build();
    }
}
```

2. **CustomerController.java** :

```java
package com.example.demo.controller;

import com.example.demo.model.Customer;
import com.example.demo.service.CustomerService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.media.Content;
import io.swagger.v3.oas.annotations.media.Schema;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.tags.Tag;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/v1/customers")
@Tag(name = "Customer", description = "API pour la gestion des clients")
public class CustomerController {

    private final CustomerService customerService;

    @Autowired
    public CustomerController(CustomerService customerService) {
        this.customerService = customerService;
    }

    @Operation(summary = "Ajouter un nouveau client", description = "Ajoute un nouveau client dans le système")
    @ApiResponse(responseCode = "200", description = "Client ajouté avec succès", content = @Content(schema = @Schema(implementation = Customer.class)))
    @ApiResponse(responseCode = "400", description = "Un client avec cet ID existe déjà")
    @PostMapping
    public ResponseEntity<Customer> addCustomer(@RequestBody Customer customer) {
        return ResponseEntity.ok(customerService.createCustomer(customer));
    }

    @Operation(summary = "Obtenir tous les clients", description = "Récupère la liste de tous les clients")
    @ApiResponse(responseCode = "200", description = "Liste des clients récupérée avec succès")
    @GetMapping
    public ResponseEntity<List<Customer>> getAllCustomers() {
        return ResponseEntity.ok(customerService.getAllCustomers());
    }

    @Operation(summary = "Obtenir un client par ID", description = "Récupère les détails d'un client spécifique")
    @ApiResponse(responseCode = "200", description = "Client trouvé", content = @Content(schema = @Schema(implementation = Customer.class)))
    @ApiResponse(responseCode = "404", description = "Client non trouvé")
    @GetMapping("/{id}")
    public ResponseEntity<Customer> getCustomerById(@Parameter(description = "ID du client") @PathVariable Long id) {
        return customerService.getCustomerById(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    @Operation(summary = "Mettre à jour un client", description = "Met à jour les détails d'un client existant")
    @ApiResponse(responseCode = "200", description = "Client mis à jour avec succès", content = @Content(schema = @Schema(implementation = Customer.class)))
    @ApiResponse(responseCode = "404", description = "Client non trouvé")
    @PutMapping("/{id}")
    public ResponseEntity<Customer> updateCustomer(@Parameter(description = "ID du client") @PathVariable Long id, @RequestBody Customer updatedCustomer) {
        return ResponseEntity.ok(customerService.updateCustomer(id, updatedCustomer));
    }

    @Operation(summary = "Supprimer un client", description = "Supprime un client spécifique s'il n'a pas de cartes associées")
    @ApiResponse(responseCode = "200", description = "Client supprimé avec succès")
    @ApiResponse(responseCode = "400", description = "Impossible de supprimer le client car il possède des cartes")
    @ApiResponse(responseCode = "404", description = "Client non trouvé")
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteCustomer(@Parameter(description = "ID du client") @PathVariable Long id) {
        customerService.deleteCustomer(id);
        return ResponseEntity.ok().build();
    }
}
```


---------------------------------
# Anciennes versions
---------------------------------

Les versions **anciennes** de vos contrôleurs géraient toute la logique au niveau du contrôleur, avec une dépendance directe entre `CustomerController` et `CardController`, et utilisaient des listes en mémoire pour stocker les entités au lieu de faire appel à une base de données.

La nouvelle version, en revanche, introduit des services et des repositories JPA pour gérer la persistance des données, ce qui améliore l'architecture de l'application en séparant les préoccupations et en permettant une interaction facile avec une base de données relationnelle.

#  comparaison entre les deux **anciens** contrôleurs (CardController et CustomerController) avec les services mis en place pour améliorer l'application.

# **ANCIEN 1 : CardController**

```java
package com.example.demo.controller;

import com.example.demo.model.Card;
import com.example.demo.model.Customer;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.media.Content;
import io.swagger.v3.oas.annotations.media.Schema;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.tags.Tag;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.context.annotation.Lazy;

import java.util.ArrayList;
import java.util.List;
import java.util.stream.Collectors;

@RestController
@RequestMapping("/api/v1/cards")
@Tag(name = "Card", description = "API pour la gestion des cartes bancaires")
public class CardController {

    private List<Card> cards = new ArrayList<>();
    private final CustomerController customerController;

    public CardController(@Lazy CustomerController customerController) {
        this.customerController = customerController;
    }

    @Operation(summary = "Ajouter une nouvelle carte", description = "Ajoute une nouvelle carte bancaire pour un client existant")
    @ApiResponse(responseCode = "200", description = "Carte ajoutée avec succès", content = @Content(schema = @Schema(implementation = Card.class)))
    @ApiResponse(responseCode = "400", description = "Données invalides ou client inexistant")
    @PostMapping
    public ResponseEntity<?> addCard(@RequestBody Card card) {
        if (cards.stream().anyMatch(c -> c.getCardId() == card.getCardId())) {
            return ResponseEntity.badRequest().body("Une carte avec cet ID existe déjà.");
        }
        if (customerController.getCustomers().stream().noneMatch(c -> c.getCustomerId() == card.getCustomerId())) {
            return ResponseEntity.badRequest().body("Le client correspondant n'existe pas.");
        }
        cards.add(card);
        return ResponseEntity.ok(card);
    }

    @Operation(summary = "Obtenir toutes les cartes", description = "Récupère la liste de toutes les cartes bancaires")
    @ApiResponse(responseCode = "200", description = "Liste des cartes récupérée avec succès")
    @GetMapping
    public List<Card> getAllCards() {
        return cards;
    }

    @Operation(summary = "Obtenir une carte par ID", description = "Récupère les détails d'une carte bancaire spécifique")
    @ApiResponse(responseCode = "200", description = "Carte trouvée", content = @Content(schema = @Schema(implementation = Card.class)))
    @ApiResponse(responseCode = "404", description = "Carte non trouvée")
    @GetMapping("/{id}")
    public ResponseEntity<?> getCardById(@Parameter(description = "ID de la carte") @PathVariable int id) {
        Card card = cards.stream()
                    .filter(c -> c.getCardId() == id)
                    .findFirst()
                    .orElse(null);
        if (card == null) {
            return ResponseEntity.notFound().build();
        }
        return ResponseEntity.ok(card);
    }

    @Operation(summary = "Mettre à jour une carte", description = "Met à jour les détails d'une carte bancaire existante")
    @ApiResponse(responseCode = "200", description = "Carte mise à jour avec succès", content = @Content(schema = @Schema(implementation = Card.class)))
    @ApiResponse(responseCode = "400", description = "Données invalides ou client inexistant")
    @ApiResponse(responseCode = "404", description = "Carte non trouvée")
    @PutMapping("/{id}")
    public ResponseEntity<?> updateCard(@Parameter(description = "ID de la carte") @PathVariable int id, @RequestBody Card updatedCard) {
        for (Card card : cards) {
            if (card.getCardId() == id) {
                if (customerController.getCustomers().stream().noneMatch(c -> c.getCustomerId() == updatedCard.getCustomerId())) {
                    return ResponseEntity.badRequest().body("Le client correspondant n'existe pas.");
                }
                card.setCustomerId(updatedCard.getCustomerId());
                card.setCardNumber(updatedCard.getCardNumber());
                card.setCardType(updatedCard.getCardType());
                card.setTotalLimit(updatedCard.getTotalLimit());
                card.setAmountUsed(updatedCard.getAmountUsed());
                card.setAvailableAmount(updatedCard.getAvailableAmount());
                card.setCreateDt(updatedCard.getCreateDt());
                return ResponseEntity.ok(card);
            }
        }
        return ResponseEntity.notFound().build();
    }

    @Operation(summary = "Supprimer une carte", description = "Supprime une carte bancaire spécifique")
    @ApiResponse(responseCode = "200", description = "Carte supprimée avec succès")
    @ApiResponse(responseCode = "404", description = "Carte non trouvée")
    @DeleteMapping("/{id}")
    public ResponseEntity<?> deleteCard(@Parameter(description = "ID de la carte") @PathVariable int id) {
        boolean removed = cards.removeIf(card -> card.getCardId() == id);
        if (removed) {
            return ResponseEntity.ok("Carte supprimée avec succès.");
        } else {
            return ResponseEntity.notFound().build();
        }
    }

    @Operation(summary = "Supprimer toutes les cartes", description = "Supprime toutes les cartes bancaires")
    @ApiResponse(responseCode = "200", description = "Toutes les cartes ont été supprimées")
    @DeleteMapping
    public ResponseEntity<String> deleteAllCards() {
        cards.clear();
        return ResponseEntity.ok("Toutes les cartes ont été supprimées.");
    }

    // Méthode utilitaire pour CustomerController
    public List<Card> getCardsByCustomerId(int customerId) {
        return cards.stream()
                    .filter(card -> card.getCustomerId() == customerId)
                    .collect(Collectors.toList());
    }
}
```

---

# **ANCIEN 2 : CustomerController**

```java
package com.example.demo.controller;

import com.example.demo.model.Customer;
import com.example.demo.model.Card;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.media.Content;
import io.swagger.v3.oas.annotations.media.Schema;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.tags.Tag;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.ArrayList;
import java.util.List;

@RestController
@RequestMapping("/api/v1/customers")
@Tag(name = "Customer", description = "API pour la gestion des clients")
public class CustomerController {

    private List<Customer> customers = new ArrayList<>();
    private final CardController cardController;

    public CustomerController(CardController cardController) {
        this.cardController = cardController;
    }

    @Operation(summary = "Ajouter un nouveau client", description = "Ajoute un nouveau client dans le système")
    @ApiResponse(responseCode = "200", description = "Client ajouté avec succès", content = @Content(schema = @Schema(implementation = Customer.class)))
    @ApiResponse(responseCode = "400", description = "Un client avec cet ID existe déjà")
    @PostMapping
    public ResponseEntity<?> addCustomer(@RequestBody Customer customer) {
        if (customers.stream().anyMatch(c -> c.getCustomerId() == customer.getCustomerId())) {
            return ResponseEntity.badRequest().body("Un client avec cet ID existe déjà.");
        }
        customers.add(customer);
        return ResponseEntity.ok(customer);
    }

    @Operation(summary = "Obtenir tous les clients", description = "Récupère la liste de tous les clients")
    @ApiResponse(responseCode = "200", description = "Liste des clients récupérée avec succès")
    @GetMapping
    public List<Customer> getAllCustomers() {
        return customers;
    }

    @Operation(summary = "Obtenir un client par ID", description = "Récupère les détails d'un client spécifique")
    @ApiResponse(responseCode = "200", description = "Client trouvé", content = @Content(schema = @Schema(implementation = Customer.class)))
    @ApiResponse(responseCode = "404", description = "Client non trouvé")
    @GetMapping("/{id}")
    public ResponseEntity<?> getCustomerById(@Parameter(description = "ID du client") @PathVariable int id) {
        Customer customer = customers.stream()
                        .filter(c -> c.getCustomerId() == id)
                        .findFirst()
                        .orElse(null);
        if (customer == null) {
            return ResponseEntity.notFound().build();
        }
        return ResponseEntity.ok(customer);
    }

    @Operation(summary = "Mettre à jour un client", description = "Met à jour les détails d'un client existant")
    @ApiResponse(responseCode = "200", description = "Client mis à jour avec succès", content = @Content(schema = @Schema(implementation = Customer.class)))
    @ApiResponse(responseCode = "404", description = "Client non trouvé")
    @PutMapping("/{id}")
    public ResponseEntity<?> updateCustomer(@Parameter(description = "ID du client") @PathVariable int id, @RequestBody Customer updatedCustomer) {
        for (Customer customer : customers) {
            if (customer.getCustomerId() == id) {
                customer.setCustomerLastName(updatedCustomer.getCustomerLastName());
                customer.setCustomerFirstName(updatedCustomer.getCustomerFirstName());
                return ResponseEntity.ok(customer);
            }
        }
        return ResponseEntity.notFound().build();
    }

    @Operation(summary =

 "Supprimer un client", description = "Supprime un client spécifique s'il n'a pas de cartes associées")
    @ApiResponse(responseCode = "200", description = "Client supprimé avec succès")
    @ApiResponse(responseCode = "400", description = "Impossible de supprimer le client car il possède des cartes")
    @ApiResponse(responseCode = "404", description = "Client non trouvé")
    @DeleteMapping("/{id}")
    public ResponseEntity<?> deleteCustomer(@Parameter(description = "ID du client") @PathVariable int id) {
        List<Card> customerCards = cardController.getCardsByCustomerId(id);
        if (!customerCards.isEmpty()) {
            return ResponseEntity.badRequest().body("Impossible de supprimer le client. Il possède encore des cartes.");
        }

        boolean removed = customers.removeIf(customer -> customer.getCustomerId() == id);
        if (removed) {
            return ResponseEntity.ok("Client supprimé avec succès.");
        } else {
            return ResponseEntity.notFound().build();
        }
    }

    @Operation(summary = "Supprimer tous les clients", description = "Supprime tous les clients s'ils n'ont pas de cartes associées")
    @ApiResponse(responseCode = "200", description = "Tous les clients ont été supprimés")
    @ApiResponse(responseCode = "400", description = "Impossible de supprimer tous les clients car certains possèdent des cartes")
    @DeleteMapping
    public ResponseEntity<String> deleteAllCustomers() {
        if (cardController.getAllCards().isEmpty()) {
            customers.clear();
            return ResponseEntity.ok("Tous les clients ont été supprimés.");
        } else {
            return ResponseEntity.badRequest().body("Impossible de supprimer tous les clients. Certains possèdent encore des cartes.");
        }
    }

    // Méthode utilitaire pour CardController
    public List<Customer> getCustomers() {
        return customers;
    }
}
```










--------------------
# **Annexe 3-2 - Ajout des packages service et exception**
--------------------

### **Annotations Swagger**

```java
/* 
Ces annotations Swagger fournissent une documentation détaillée pour chaque endpoint de l'API des clients, y compris :
- Une description générale de l'API des clients (@Tag).
- Des résumés et descriptions pour chaque opération (@Operation).
- Des détails sur les réponses possibles pour chaque endpoint (@ApiResponse).
- Des descriptions pour les paramètres de chemin (@Parameter).

Cette documentation permettra aux développeurs d'avoir une meilleure compréhension de votre API et facilitera son utilisation.
*/
```

--------------------
# **Annexe 4 - Ajout des packages service et exception**
--------------------

Voici les implémentations des services pour **Customer** et **Card**. Ces services implémentent la logique métier et interagissent avec les repositories JPA.

---

1. **CustomerService.java** :

```java
package com.example.demo.service;

import com.example.demo.model.Customer;
import com.example.demo.repository.CustomerRepository;
import com.example.demo.repository.CardRepository;
import com.example.demo.exception.ResourceNotFoundException;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.Optional;

@Service
@Transactional
public class CustomerService {

    private final CustomerRepository customerRepository;
    private final CardRepository cardRepository;

    @Autowired
    public CustomerService(CustomerRepository customerRepository, CardRepository cardRepository) {
        this.customerRepository = customerRepository;
        this.cardRepository = cardRepository;
    }

    public Customer createCustomer(Customer customer) {
        return customerRepository.save(customer);
    }

    public List<Customer> getAllCustomers() {
        return customerRepository.findAll();
    }

    public Optional<Customer> getCustomerById(Long id) {
        return customerRepository.findById(id);
    }

    public Customer updateCustomer(Long id, Customer customerDetails) {
        Customer customer = customerRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Customer not found with id: " + id));
        
        customer.setCustomerFirstName(customerDetails.getCustomerFirstName());
        customer.setCustomerLastName(customerDetails.getCustomerLastName());
        
        return customerRepository.save(customer);
    }

    public void deleteCustomer(Long id) {
        Customer customer = customerRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Customer not found with id: " + id));
        
        if (!customer.getCards().isEmpty()) {
            throw new IllegalStateException("Cannot delete customer with active cards");
        }
        
        customerRepository.delete(customer);
    }

    public void deleteAllCustomers() {
        if (cardRepository.count() > 0) {
            throw new IllegalStateException("Cannot delete all customers. Some customers have active cards.");
        }
        customerRepository.deleteAll();
    }

    public List<Customer> findCustomersByName(String name) {
        return customerRepository.findByCustomerLastNameContainingOrCustomerFirstNameContaining(name, name);
    }
}
```

---

2. **CardService.java** :

```java
package com.example.demo.service;

import com.example.demo.model.Card;
import com.example.demo.model.Customer;
import com.example.demo.repository.CardRepository;
import com.example.demo.repository.CustomerRepository;
import com.example.demo.exception.ResourceNotFoundException;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.Optional;

@Service
@Transactional
public class CardService {

    private final CardRepository cardRepository;
    private final CustomerRepository customerRepository;

    @Autowired
    public CardService(CardRepository cardRepository, CustomerRepository customerRepository) {
        this.cardRepository = cardRepository;
        this.customerRepository = customerRepository;
    }

    public Card createCard(Card card, Long customerId) {
        Customer customer = customerRepository.findById(customerId)
            .orElseThrow(() -> new ResourceNotFoundException("Customer not found with id: " + customerId));
        card.setCustomer(customer);
        return cardRepository.save(card);
    }

    public List<Card> getAllCards() {
        return cardRepository.findAll();
    }

    public Optional<Card> getCardById(Long id) {
        return cardRepository.findById(id);
    }

    public Card updateCard(Long id, Card cardDetails) {
        Card card = cardRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Card not found with id: " + id));
        
        card.setCardNumber(cardDetails.getCardNumber());
        card.setCardType(cardDetails.getCardType());
        card.setTotalLimit(cardDetails.getTotalLimit());
        card.setAmountUsed(cardDetails.getAmountUsed());
        card.setAvailableAmount(cardDetails.getAvailableAmount());
        card.setCreateDt(cardDetails.getCreateDt());
        
        return cardRepository.save(card);
    }

    public void deleteCard(Long id) {
        Card card = cardRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Card not found with id: " + id));
        cardRepository.delete(card);
    }

    public void deleteAllCards() {
        cardRepository.deleteAll();
    }

    public List<Card> getCardsByCustomerId(Long customerId) {
        return cardRepository.findByCustomerId(customerId);
    }

    public Optional<Card> findCardByNumber(String cardNumber) {
        return cardRepository.findByCardNumber(cardNumber);
    }
}
```

---

Ces services implémentent toutes les opérations CRUD de base, ainsi que quelques méthodes spécifiques. Ils utilisent les repositories JPA pour interagir avec la base de données et gèrent les exceptions pour les ressources non trouvées.



--------------------
# **Annexe 5 - Ajout des repositories**
--------------------

Voici les interfaces repository pour les entités **Customer** et **Card**, qui étendent **JpaRepository** pour bénéficier des méthodes CRUD de base :

1. **CustomerRepository.java** :

```java
package com.example.demo.repository;

import com.example.demo.model.Customer;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public interface CustomerRepository extends JpaRepository<Customer, Long> {

    // Recherche des clients par nom (prénom ou nom de famille)
    List<Customer> findByCustomerLastNameContainingOrCustomerFirstNameContaining(String lastName, String firstName);
}
```

---

2. **CardRepository.java** :

```java
package com.example.demo.repository;

import com.example.demo.model.Card;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Optional;

@Repository
public interface CardRepository extends JpaRepository<Card, Long> {

    // Recherche des cartes par ID du client
    List<Card> findByCustomerId(Long customerId);

    // Recherche d'une carte par numéro de carte
    Optional<Card> findByCardNumber(String cardNumber);
}
```

Ces interfaces permettent les opérations CRUD de base et incluent des méthodes de recherche personnalisées.


--------------------
# **Annexe 6 - Correction des erreurs**
--------------------

L'erreur rencontrée concernant la méthode `createCard` dans **CardService**, qui attend deux arguments, se résout en passant correctement l'ID du client. Voici comment procéder :

### Correction du code :

#### **CardService.java** :

```java
@Service
@Transactional
public class CardService {

    private final CardRepository cardRepository;
    private final CustomerRepository customerRepository;

    @Autowired
    public CardService(CardRepository cardRepository, CustomerRepository customerRepository) {
        this.cardRepository = cardRepository;
        this.customerRepository = customerRepository;
    }

    public Card createCard(Card card, Long customerId) {
        Customer customer = customerRepository.findById(customerId)
            .orElseThrow(() -> new ResourceNotFoundException("Customer not found"));
        card.setCustomer(customer);
        return cardRepository.save(card);
    }
}
```

#### **CardController.java** :

```java
@RestController
@RequestMapping("/api/v1/cards")
@Tag(name = "Card", description = "API pour la gestion des cartes bancaires")
public class CardController {

    private final CardService cardService;

    @Autowired
    public CardController(CardService cardService) {
        this.cardService = cardService;
    }

    @Operation(summary = "Ajouter une nouvelle carte", description = "Ajoute une nouvelle carte bancaire pour un client existant")
    @ApiResponse(responseCode = "200", description = "Carte ajoutée avec succès", content = @Content(schema = @Schema(implementation = Card.class)))
    @ApiResponse(responseCode = "400", description = "Données invalides ou client inexistant")
    @PostMapping
    public ResponseEntity<Card> addCard(@RequestBody Card card, @RequestParam Long customerId) {
        return ResponseEntity.ok(cardService.createCard(card, customerId));
    }
}
```



















