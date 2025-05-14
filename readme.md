## Aperçu du projet

Ce projet propose un **système de planning générique** pour toute organisation (entreprise, hôpital, administration…), permettant de gérer utilisateurs, tâches, départements et plannings de manière découplée et extensible.
Il intègre deux **design patterns** de création GoF :

* **Factory Method** pour centraliser la création d’entités métier.
* **Builder** pour assembler de façon fluide des objets complexes (tâches, plannings, entrées de planning).

---

## Table des matières

1. [Diagramme de classes global](#diagramme-de-classes-global)
2. [Design patterns utilisés](#design-patterns-utilisés)

    * [Factory Method](#factory-method)
    * [Builder](#builder)
3. [Logique métier & workflow de création](#logique-métier--workflow-de-création)
4. [Étapes de mise en place](#étapes-de-mise-en-place)
5. [Installation & déploiement](#installation--déploiement)

---

## Diagramme de classes global

```
                                    +-----------------------+
                                    |      <<interface>>    |
                                    |        Entity         |
                                    +-----------------------+
                                    | +getId(): Long        |
                                    +-----------------------+

                                             ^
                                             |
             +-------------------+            |             +-------------------+
             |   AbstractFactory |            |             |   AbstractBuilder |
             +-------------------+            |             +-------------------+
             | +create(type:String):Entity    |             | +build():Entity   |
             +-------------------+            |             +-------------------+
                       ^                        \___________|                   |
                       |                                    |                   |
      +------------------------+                    +----------------+  +------------------+
      |     EntityFactory      |                    | TaskBuilder    |  | PlanningBuilder  |
      +------------------------+                    +----------------+  +------------------+
      | +create(type:String)   |                    | -title:String  |  | -period:Period   |
      +------------------------+                    | -desc:String   |  | -entries:List<>  |
                                                    | -start:Date    |  +------------------+
                                                    | -end:Date      |  | +addEntry(e):    |
                                                    | +set...()      |  |   PlanningBuilder|
                                                    | +build():Task  |  | +build():Planning|
                                                    +----------------+  +------------------+

+-----------------+   *     +-------------+    *    +----------------+
|      User       |--------|    Task     |--------|   Department   |
+-----------------+        +-------------+        +----------------+
| -id: Long       |        | -id: Long   |        | -id: Long       |
| -name: String   |        | -title:String|       | -name:String    |
| -role: String   |        | -desc:String|        +----------------+
+-----------------+        | -start:Date |        
        ^                  | -end:Date   |        
        |                  | -status:Enum|        
        |                  +-------------+        
        |                         ^              
        |                         |              
        |                  +--------------+      
        |                  |    Entry     |      
        |                  +--------------+      
        |                  | -id: Long    |      
        |                  | -user:User   |      
        |                  | -task:Task   |      
        |                  | -period:Period|     
        |                  +--------------+      
        |                         ^              
        |                         |              
        |                +----------------------+
        |                |      Planning        |
        |                +----------------------+
        |                | -id: Long            |
        +--------------->| -period: Period      |
                         | -entries: List<Entry>|
                         +----------------------+
```

---

## Design patterns utilisés

### Factory Method

* **Objectif** : Déléguer la création d’objets à une factory pour éviter les `new` dispersés et le couplage au type concret.
* **Classe clé** : `EntityFactory` (sous-classe de `AbstractFactory`), méthode `create(String type)`.
* **Avantages** :

    * Centralisation de la logique d’instanciation.
    * Extensibilité : ajouter un nouveau type se fait en un seul `case`.
    * Testabilité : factory mockable en tests unitaires.

### Builder

* **Objectif** : Construire des objets complexes (avec 4–5 paramètres ou plus) de façon fluide et sûre.
* **Classes clés** :

    * `TaskBuilder` → `Task`
    * `PlanningBuilder` → `Planning`
* **Avantages** :

    * API enchaînée, plus lisible que les constructeurs à longs paramètres.
    * Validation possible (`start ≤ end`, etc.) avant `build()`.
    * Séparation claire entre construction et représentation.

---

## Logique métier & workflow de création

1. **Initialisation du contexte**

    * Chargement des configurations (DB, JPA, Spring Boot).
    * Déclaration des factories et builders en tant que beans Spring.

2. **Création d’une entité (User, Task, Department)**

    * Appel à `EntityFactory.create("user")` ou `"task"`, sans connaître la classe concrète.

3. **Construction d’un objet complexe (Tâche)**

   ```java
   Task t = new TaskBuilder("Réunion")
                .description("Sync mensuelle")
                .start(startDate)
                .end(endDate)
                .build();
   ```

4. **Construction d’un planning**

   ```java
   Planning p = new PlanningBuilder(period)
                     .addEntry(new Entry(user, task, assignPeriod))
                     .addEntry(…)
                     .build();
   ```

5. **Persistance & services**

    * Entities persistées via `JpaRepository`.
    * Services métier orchestrent la création, la validation et la mise à jour des plannings.

6. **Exposition & consommation**

    * **REST API** via Spring (`@RestController`).
    * **Front-end** (React / Vue / Angular) consomme les endpoints pour afficher un calendrier/Gantt et gérer CRUD.

---

## Étapes de mise en place

1. **Cloner le dépôt**

   ```bash
   git clone https://votre-repo/erp-planning.git
   cd erp-planning
   ```

2. **Back-end (Java / Spring Boot)**

    * Configurer `application.properties` pour la base de données.
    * Lancer : `./mvnw spring-boot:run` ou `./gradlew bootRun`.

3. **Front-end (React)**

   ```bash
   cd frontend
   npm install
   npm start
   ```

    * L’interface démarre sur `http://localhost:3000` et communique avec l’API.

4. **Tests**

    * Back-end : `./mvnw test`
    * Front-end : `npm test`

5. **Build & déploiement**

    * Back-end : `./mvnw package` → JAR/Docker
    * Front-end : `npm run build` → fichiers statiques (servir via Nginx ou Spring Boot)

---

## Developpement

## 1. Mise en place de l’environnement

1. **Initialiser le dépôt Git et la CI**

    * *Pourquoi* : versionner le code dès le départ et automatiser les tests/intégration continue.
2. **Configurer le projet Spring Boot (backend)**

    * Créer un projet via Spring Initializr avec dépendances Web, JPA, Security.
    * *Pourquoi* : disposer immédiatement d’un squelette REST, d’un ORM et d’une base pour la sécurité.
3. **Configurer le projet React (frontend)**

    * `npx create-react-app`, installer Axios et la librairie calendrier (FullCalendar ou équivalent).
    * *Pourquoi* : démarrer l’interface utilisateur et préparer la consommation des APIs.

## 2. Modélisation des données et patterns de création

1. **Définir les entités JPA** (`User`, `Task`, `Department`, `Entry`, `Planning`)

    * Annoter chaque classe avec `@Entity` et configurer les relations (`@ManyToOne`, `@OneToMany`).
    * *Pourquoi* : fixer le cœur du modèle métier et préparer la persistance.
2. **Implémenter le Factory Method**

    * Créer `AbstractFactory` et `EntityFactory.create(type)` pour `User`, `Task`, `Department`.
    * *Pourquoi* : centraliser et découpler la création d’objets métier.
3. **Implémenter les Builders**

    * `TaskBuilder` pour assembler les tâches (titre, dates, département).
    * `PlanningBuilder` pour composer un planning via une liste d’`Entry`.
    * *Pourquoi* : garantir un état cohérent et lisible à la construction des objets complexes.

## 3. Logique métier et services

1. **Développer les services métier** (`@Service`)

    * Règles de non-chevauchement, transitions de statut, calcul de charge.
    * Utiliser `@Transactional` pour la cohérence.
    * *Pourquoi* : isoler la logique métier cruciale du reste de l’application.
2. **Écrire les exceptions métier**

    * `OverlapException`, `InvalidStatusException`, etc.
    * *Pourquoi* : gérer proprement les erreurs et remonter des messages clairs à l’API.

## 4. API REST et contrôle d’accès

1. **Créer les contrôleurs** (`@RestController`)

    * Endpoints CRUD pour chaque entité et endpoints métier pour les plannings.
    * *Pourquoi* : exposer la logique métier au frontend.
2. **Configurer Spring Security**

    * Authentification JWT, rôles (ADMIN, MANAGER, USER).
    * *Pourquoi* : sécuriser l’accès et les opérations selon les profils.

## 5. Front-end et interfaces utilisateur

1. **Construire les formulaires**

    * `UserForm`, `TaskForm`, `DepartmentForm`, `PlanningForm`.
    * Validation basique des champs (champs obligatoires, format dates).
    * *Pourquoi* : guider l’utilisateur et prévenir les erreurs simples avant envoi.
2. **Intégrer le calendrier/Gantt**

    * Utiliser FullCalendar ou React-Gantt pour afficher les entrées de planning.
    * Mapper les `Entry` reçues de l’API en événements.
    * *Pourquoi* : offrir une vue intuitive et interactive du planning.

## 6. Tests et qualité

1. **Tests unitaires backend**

    * `Factory`, `Builder`, `Service` et exceptions.
    * *Pourquoi* : garantir le bon fonctionnement de la logique métier.
2. **Tests d’intégration**

    * Lancement en mémoire de la base et tests des controllers.
    * *Pourquoi* : valider l’ensemble du flux API–base.
3. **Tests end-to-end frontend**

    * Avec Cypress ou Selenium pour un scénario de bout en bout (création d’une tâche, affectation, affichage).
    * *Pourquoi* : s’assurer que le front communique correctement avec le back et que l’UX est fonctionnelle.

## 7. Déploiement

1. **Containerisation Docker**

    * Dockerfile pour le backend et le frontend.
    * *Pourquoi* : faciliter les déploiements cohérents et reproductibles.
2. **Configuration CI/CD**

    * Pipeline GitHub Actions / GitLab CI pour builder, tester et déployer.
    * *Pourquoi* : automatiser le cycle de vie et réduire les erreurs manuelles.

---
