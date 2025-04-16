### **Examen Pratique : Application de Location de Voiture**

---

### **Introduction**
Cet examen pratique est conçu pour évaluer vos compétences en développement, tests unitaires, tests d'intégration, tests E2E (end-to-end) et en mise en œuvre d'outils de qualité du code comme SonarQube et JMeter. Vous travaillerez sur une application de **location de voiture** développée en Java avec Spring Boot. L'examen est divisé en plusieurs parties avec des objectifs clairs.

---

### **Objectifs**

1. Implémenter des **tests unitaires**, des **tests avec mocks et spies**, et des **tests d'intégration** pour une application existante.
2. Ajouter **deux nouvelles fonctionnalités** selon le principe du **Test-Driven Development (TDD)** et écrire les tests nécessaires.
3. Implémenter **3 scénarios E2E** en utilisant **Cucumber**.
4. Pluger SonarQube sur le projet et fournir un conteneur Docker pour l'exécuter localement.
5. Créer un fichier `.jmx` pour effectuer des tests de charge avec **JMeter**.

---

### **Instructions Générales**

1. **Prérequis logiciels :**
   - **Java 11** ou plus récent
   - **Maven** ou **Gradle**
   - **Docker** (pour SonarQube)
   - **JMeter** (pour les tests de charge)

2. **Livrables attendus :**
   - Code source complet avec les tests unitaires, mocks, spies, et tests d'intégration.
   - Code des deux nouvelles fonctionnalités avec leurs tests (TDD).
   - Fichier `cucumber.feature` pour les scénarios E2E.
   - Fichier Docker Compose pour exécuter SonarQube.
   - Fichier `.jmx` pour les tests de charge.

---

## **Partie 1 : Application de Location de Voiture**

### **Code de Base**

Les classes suivantes sont fournies. Vous devez écrire les tests unitaires, les tests avec mocks et spies, ainsi que les tests d'intégration.

#### Classe `Car`
```java name=src/main/java/com/example/rental/Car.java
package com.example.rental;

public class Car {
    private String registrationNumber;
    private String model;
    private boolean available;

    public Car(String registrationNumber, String model, boolean available) {
        this.registrationNumber = registrationNumber;
        this.model = model;
        this.available = available;
    }

    public String getRegistrationNumber() {
        return registrationNumber;
    }

    public String getModel() {
        return model;
    }

    public boolean isAvailable() {
        return available;
    }

    public void setAvailable(boolean available) {
        this.available = available;
    }
}
```

---

#### Classe `CarRepository`
```java name=src/main/java/com/example/rental/CarRepository.java
package com.example.rental;

import org.springframework.stereotype.Repository;

import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

@Repository
public class CarRepository {
    private final List<Car> cars = new ArrayList<>();

    public List<Car> getAllCars() {
        return cars;
    }

    public Optional<Car> findByRegistrationNumber(String registrationNumber) {
        return cars.stream()
                .filter(car -> car.getRegistrationNumber().equals(registrationNumber))
                .findFirst();
    }

    public void addCar(Car car) {
        cars.add(car);
    }

    public void updateCar(Car car) {
        cars.replaceAll(c -> c.getRegistrationNumber().equals(car.getRegistrationNumber()) ? car : c);
    }
}
```

---

#### Service `CarRentalService`
```java name=src/main/java/com/example/rental/CarRentalService.java
package com.example.rental;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

@Service
public class CarRentalService {

    @Autowired
    private CarRepository carRepository;

    public List<Car> getAllCars() {
        return carRepository.getAllCars();
    }

    public boolean rentCar(String registrationNumber) {
        Optional<Car> car = carRepository.findByRegistrationNumber(registrationNumber);
        if (car.isPresent() && car.get().isAvailable()) {
            car.get().setAvailable(false);
            carRepository.updateCar(car.get());
            return true;
        }
        return false;
    }

    public void returnCar(String registrationNumber) {
        Optional<Car> car = carRepository.findByRegistrationNumber(registrationNumber);
        car.ifPresent(c -> {
            c.setAvailable(true);
            carRepository.updateCar(c);
        });
    }
}
```

---

#### Contrôleur `CarController`
```java name=src/main/java/com/example/rental/CarController.java
package com.example.rental;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/cars")
public class CarController {

    @Autowired
    private CarRentalService carRentalService;

    @GetMapping
    public List<Car> getAllCars() {
        return carRentalService.getAllCars();
    }

    @PostMapping("/rent/{registrationNumber}")
    public boolean rentCar(@PathVariable String registrationNumber) {
        return carRentalService.rentCar(registrationNumber);
    }

    @PostMapping("/return/{registrationNumber}")
    public void returnCar(@PathVariable String registrationNumber) {
        carRentalService.returnCar(registrationNumber);
    }
}
```

---

### **Travail demandé - Partie 1**

1. **Tests unitaires :**
   - Écrire des tests unitaires pour chaque méthode des classes `CarRepository` et `CarRentalService`.
   - Utiliser des mocks pour isoler les dépendances (par exemple, mocker `CarRepository` dans `CarRentalService`).

2. **Tests avec spies :**
   - Ajouter des tests pour vérifier les interactions entre les différentes méthodes (par exemple, vérifier que `updateCar` est bien appelée après `rentCar`).

3. **Tests d'intégration :**
   - Écrire des tests pour les endpoints REST dans `CarController`.
   - Utiliser MockMvc pour simuler les requêtes HTTP.

---

## **Partie 2 : Nouvelles Fonctionnalités avec TDD**

Ajoutez les deux fonctionnalités suivantes :

1. **Ajouter une voiture :**
   - Endpoint `POST /cars/add` pour ajouter une nouvelle voiture.
   - La voiture ne peut être ajoutée que si son numéro d'immatriculation est unique.

2. **Rechercher une voiture par modèle :**
   - Endpoint `GET /cars/search?model={model}` pour rechercher toutes les voitures d'un modèle donné.

### **Travail demandé - Partie 2**

1. Suivez les principes du **Test-Driven Development (TDD)** :
   - Écrivez les tests avant d'implémenter les fonctionnalités.
   - Testez les scénarios de succès et d'échec (par exemple, tentative d'ajouter une voiture avec un numéro d'immatriculation déjà existant).

2. Implémentez les fonctionnalités une fois les tests écrits.

---

## **Partie 3 : Tests E2E avec Cucumber**

Écrivez **3 scénarios de tests E2E** pour l'application en utilisant **Cucumber** :

1. **Scénario : Lister toutes les voitures disponibles**
   ```gherkin
   Given des voitures sont disponibles
   When je demande la liste des voitures
   Then toutes les voitures sont affichées
   ```

2. **Scénario : Louer une voiture avec succès**
   ```gherkin
   Given une voiture est disponible
   When je loue cette voiture
   Then la voiture n'est plus disponible
   ```

3. **Scénario : Retourner une voiture**
   ```gherkin
   Given une voiture est louée
   When je retourne cette voiture
   Then la voiture est marquée comme disponible
   ```

---

## **Partie 4 : SonarQube**

1. **Pluger SonarQube :**
   - Ajoutez le plugin **SonarQube** dans votre projet Maven ou Gradle.
   - Fournissez un fichier Docker Compose pour exécuter SonarQube localement.

   Exemple de fichier `docker-compose.yml` :
   ````yaml name=docker-compose.yml
   version: "3"
   services:
     sonarqube:
       image: sonarqube:community
       container_name: sonarqube
       ports:
         - "9000:9000"
       environment:
         SONAR_ES_BOOTSTRAP_CHECKS_DISABLE: "true"
   ````

2. Fournissez un rapport SonarQube montrant la couverture des tests et les éventuels problèmes détectés.

---

## **Partie 5 : Tests de Charge avec JMeter**

1. Créez un fichier `.jmx` pour exécuter une batterie de tests de charge :
   - **Scénario 1** : 100 utilisateurs simultanés demandent la liste des voitures (`GET /cars`).
   - **Scénario 2** : 50 utilisateurs simultanés louent des voitures (`POST /cars/rent/{registrationNumber}`).
   - **Scénario 3** : 30 utilisateurs retournent des voitures (`POST /cars/return/{registrationNumber}`).

2. Fournissez les métriques collectées (temps de réponse, taux d'erreur, etc.).

---

### **Livrables**

1. Code source complet (tests unitaires, mocks, spies, intégration, TDD).
2. Fichiers `.feature` pour les scénarios Cucumber.
3. Rapport SonarQube.
4. Fichier dockerfile and `docker-compose.yml` (optional) pour SonarQube. 
5. Fichier `.jmx` pour les tests de charge. (optional) 


---

### **Ressources utiles**

- [Documentation Spring Boot](https://spring.io/projects/spring-boot)
- [Mockito Documentation](https://site.mockito.org/)
- [Cucumber Documentation](https://cucumber.io/docs/)
- [SonarQube Documentation](https://docs.sonarqube.org/)
- [JMeter Documentation](https://jmeter.apache.org/)

---

Bonne chance pour cet examen ! 😊
