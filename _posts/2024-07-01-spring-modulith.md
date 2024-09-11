---
layout: post
title: "Spring Modulith ou comment redonner vie à l'Architecture Monolithique"
resume: "Le sujet de cet article est de montrer que l'Architecture Monolithique Modulaire peut offrir un compromis 
élégant entre la simplicité de l'architecture Monolithique, le découplage du Domain-Driven-Design et la 
modularité des architectures distribuées."
image: "/assets/images/spring-modulith.jpg"
readTime: "15"
excerpt: Spring Modulith monolithique monolithic modularity modulaire architecture hexagonale events DDD java
---

## Table des matières <!-- omit in toc -->

1. [Introduction](#introduction)
2. [L'Architecture Monolithique Modulaire](#larchitecture-monolithique-modulaire)
     1. [Un peu d'histoire](#un-peu-dhistoire)
     2. [Concrètement, qu'est-ce qu'une Architecture Monolithique Modulaire ?](#concrètement-quest-ce-quune-architecture-monolithique-modulaire-)
3. [Le besoin](#le-besoin)
4. [Spring Modulith à la rescousse](#spring-modulith-à-la-rescousse)
   1. [Dépendances](#dépendances)
   2. [Découpage en modules](#découpage-en-modules)
   3. [Couplage inter-modules](#couplage-inter-modules)
      1. [Injection de dépendance non autorisée](#injection-de-dépendance-non-autorisée)
      2. [Utilisation de type non autorisé](#utilisation-de-type-non-autorisé)
   4. [Couplage intra-module](#couplage-intra-module)
   5. [L'Event-Driven selon Spring Modulith](#levent-driven-selon-spring-modulith)
   6. [Documentation](#documentation)
   7. [Modulariser et isoler ses tests](#modulariser-et-isoler-ses-tests)
5. [Quoi d'autre ?](#quoi-dautre-)
6. [Convaincus ?](#convaincus-)
7. [Liens Utiles](#liens-utiles)

## Introduction

Monolithique, Client-Serveur, N-Tiers, Microservices, SOA, Hexagonale, MVC, Event-Driven... les architectures logicielles sont nombreuses et souvent combinées au sein d'une même application.

Depuis des années, de plus en plus d'entreprises se tournent vers les architectures Hexagonales, Domain-Driven, Event-Driven ou Microservices (voire un mix de plusieurs d'entre elles), délaissant ainsi leurs vieilles architectures Monolithiques, moins vendeuses, moins évolutives, moins scalable et avouons-le, moins fun à développer.

L'architecture dite **Hexagonale** (ou *Ports/Adapters*) met l'accent sur la séparation des préoccupations (*separation of concerns*) en définissant clairement les interfaces externes (appelées "ports") et en utilisant leurs implémentations (appelées "adaptateurs") pour connecter ces interfaces au noyau métier.

L'architecture dite **Domain-Driven-Design** (ou *DDD*) est assez proche de l'architecture Hexagonale.
Mais elle repose sur la notion de *Bounded Context* (ou "*Contexte Borné*"), qui représente un contexte métier limité et isolé.

L'architecture **Microservices** est un style architectural dans lequel une application est construite comme un ensemble de petits services autonomes, chacun gérant un domaine métier spécifique.
Chaque service est déployé indépendamment et communique généralement avec les autres services via des protocoles légers (HTTP/REST, gRPC, AMQP).

L'architecture **Event-Driven** (ou *Orientée Évènements*) est un style architectural dans lequel les composants de l'application communiquent entre eux en réagissant aux événements qui se produisent dans le système. Les événements sont des messages asynchrones qui transportent des informations sur un changement d'état ou une action dans le système.

L'architecture **Monolithique** traditionnelle est un style d'architecture logicielle dans lequel toute l'application est conçue et déployée comme une seule unité. Cette dernière est généralement composée d'une seule base de code, où toutes les fonctionnalités sont développées, testées, déployées et maintenues ensemble.  
En raison de la nature intégrée de toutes les parties de l'application, il existe souvent un couplage fort entre les différents composants, d'où son nom peu flatteur d'*architecture spaghetti*.

Mais alors quelle architecture choisir pour créer un nouveau produit, si vous êtes séduits à la fois par la simplicité du monolithe, par la modularité des microservices, par l'isolation du code métier de l'architecture Hexagonale ou du Domain-Driven-Design, et par le côté asynchrone de l'Event-Driven ? 🤔

## L'Architecture Monolithique Modulaire

### Un peu d'histoire

On retrouve des traces de publications de modèles de conception encourageant la modularité dès 1994 avec par exemple le très connu "*Design Patterns: Elements of Reusable Object-Oriented Software*", livre écrit par le "*Gang of Four*" : Erich Gamma, Richard Helm, Ralph Johnson, et John Vlissides.  
Robert C. Martin, aussi connu sous le nom de "*Uncle Bob*", a formulé les principes *SOLID*, un ensemble de cinq principes de conception qui promeut la modularité et l'évolutivité des logiciels.  
Contrairement à ce que la majorité d'entre nous pourrait penser, *Spring Framework*, créé par Rod Johnson en 2003, a popularisé les concepts de modularité dans les applications Java, en facilitant la séparation des préoccupations, principalement à travers l'inversion de contrôle.

Ce type d'architecture n'est donc pas récent et n'a pas vraiment été inventé, c'est plutôt une évolution des pratiques du développement logiciel.

Nos *architectures spaghetti* ne sont-elles pas simplement dues au fait que nous avons oublié comment bien structurer une application, qu'elle soit monolithique ou non ?

### Concrètement, qu'est-ce qu'une Architecture Monolithique Modulaire ?

Contrairement aux architectures distribuées, comme les microservices ou SOA, l'**Architecture Monolithique Modulaire** simplifie le déploiement, la gestion et l'évolutivité de l'application en ne nécessitant qu'un seul environnement d'exécution.  
En divisant l'application en modules distincts, définis par leur **Bounded Context**, chaque module peut être développé, testé et maintenu de manière indépendante, ce qui favorise la ré-utilisabilité du code et la séparation des préoccupations.  
En évitant les communications réseau et en partageant les ressources au sein du même processus d'exécution, elle pourrait même offrir des performances supérieures à celles d'une architecture distribuée.  
Enfin, un des modules peut très facilement être extrait pour fonctionner de manière autonome (pour des raisons de charge ou de performances) dû au faible couplage de cette architecture.

Contrairement à l'architecture Hexagonale, qui se concentre sur la séparation du domaine Métier de l'Infrastructure, l'architecture Modulaire se concentre sur la séparation en modules métier (comme DDD). Chacun de ces modules peut être ensuite architecturé de la manière souhaitée : en couches ou de manière hexagonale.  
Le plus important est que chaque module n'expose que ses interfaces utiles (ou aucune), de manière à limiter le couplage que nous retrouvons habituellement dans l'architecture monolithique traditionnelle.

Côté interactions entre les différents modules, vous avez le choix : appels directs, évènements/messages, base de données partagées, système d'échange de fichiers...  
Le graphique ci-dessous montre que la solution hybride "appels directs et évènements/messages" est préférable. Elle simplifie les échanges, tout en limitant le couplage et en gardant des temps de synchronisation de données très acceptables.

![interactions](/assets/2024/05/24/01-couplage-complexité-synchro.png)

Si nous devions résumer, l'Architecture Monolithique Modulaire offre un **compromis** élégant entre la **simplicité** de l'architecture Monolithique, le **découplage** du Domain-Driven-Design et la **modularité** des architectures distribuées.

## Le besoin

Imaginez que vous ayez une application de gestion d'établissements scolaires à développer.  
Rapidement, vous identifiez 3 *bounded context*.

* **"établissements"** responsable de la gestion :
  * des informations des établissements scolaires (nom, adresse, contact...)
  * des employés (enseignants, agents d'entretien, administration...)
  * du planning des activités scolaires (réunions, fêtes, examens...)
* **"classes"** responsable de la gestion :
  * des classes (attribution d'enseignants, matières...)
  * des emplois du temps
* **"élèves"** responsable de la gestion :
  * des élèves (données personnelles, inscription...)
  * des absences
  * de la communication avec les parents (bulletins de notes, notification d'absence...)
  * des notes obtenues aux évaluations

Vous êtes Java-istes et/ou Kotlin-istes, et assez férus de Spring.

L'approche monolithique classique avec Spring serait :

```acme
src/main/java
    ...acme.myproject
    ...acme.myproject.controler
    ...acme.myproject.service
    ...acme.myproject.dao
```

Très vite, vous constatez que le couplage sera fort et que l'évolutivité ne sera pas aisée.

L'approche hexagonale ressemblerait à :

```acme
acme/myproject
    ...infrastructure
    ...infrastructure.src.main.java
    ...domain
    ...domain.src.main.java
    ...domain.src.main.java.module1
    ...domain.src.main.java.module1.spi
    ...domain.src.main.java.module1.api
    ...domain.src.main.java.module2
    ...domain.src.main.java.module2.spi
    ...domain.src.main.java.module2.api
```

Cette approche est très bien, mais vous avez des équipes de développements par module et elles doivent éviter de gérer du code commun, à savoir ici "*infrastructure*".

Votre choix d'architecture modulaire repose donc sur un découpage en 3 modules distincts correspondants aux 3 *bounded context* : "establishment", "classroom" et "student".  
Comment vous assurer maintenant que le couplage entre modules restera faible, voire inexistant ?  
Quel découpage par module devez-vous choisir ? Les modules proposés depuis Java 9, les modules Maven, un découpage en packages ?  
Devez-vous demander au leader technique et/ou à l'architecte logiciels une liste des bonnes pratiques, qui seront oubliées aussi vite qu'un simple tour de bocal ?

## Spring Modulith à la rescousse

Le framework [Spring Modulith](https://docs.spring.io/spring-modulith/reference/) répond à l'ensemble de ces interrogations.

Vous allez voir qu'il permet de modulariser un monolithe, de tester son architecture orientée "Modules Métier" en s'appuyant sur les librairies *ArchUnit* et *JMolecule*, de tester les modules indépendamment les uns des autres, de générer la documentation de chaque module, de communiquer de façon asynchrone entre modules, voire de gérer la "duplication" de données d'un module à un autre via des évènements (Intra-JVM, RabbitMQ, Kafka, JMS...), simplifiant ainsi énormément la solution via Messaging, et le tout en gardant un couplage très faible.

Essayons ! Essayez !

### Dépendances

Si vous partez de zéro, le plus facile est d'initialiser votre projet Spring Boot via [Spring Initializr](https://start.spring.io/), en incluant la dépendance `Spring Modulith`.  
Sinon, ajoutez `spring-modulith-bom` en tant que *dependencyManagement* et ajoutez la dépendance `spring-modulith-starter-core` à votre projet existant.  
Vous pouvez retrouver dans la [documentation officielle](https://docs.spring.io/spring-modulith/reference/appendix.html#artifacts) la liste des *Starters Spring* et les librairies qu'ils incluent.

### Découpage en modules

Avec Spring Modulith, chaque package correspond à un module.

Le package "racine" d'un module équivaut au package "*api*" de l'architecture hexagonale (interface d'entrée du domaine métier).  
Il contient l'interface `<Module>ServiceInterface.java` exposée par le domaine métier, le Controller REST `<Module>Controller.java` permettant l'accès au service métier par des applications externes, et un sous-package (appelons-le `internal`) dans lequel se trouve le code métier, les entités et la persistence des données.

À la racine de chaque module, vous retrouvez le sous-package `spi` contenant les interfaces de sortie du domaine métier (émission d'évènements, appel de services exposés par d'autres modules, appels de services externes...), les DTO et les mappers.

Garder un domaine métier le moins dépendant possible est réalisable. Dans la structure ci-dessous, le sous-package `spi` contient également un sous-package `external` dans lequel vous aurez l'implémentation de l'interface `ExternalInterface.java`. Celle-ci permet d'appeler des services d'autres modules (via l'injection de dépendances) et donc au final, de n'avoir au sein de votre service métier que des dépendances internes, vers le(s) Repository(s) du module, et vers les interfaces du package `spi`.  
Le couplage du domaine métier est ainsi fortement limité, et si un des modules est externalisé quelque temps plus tard, seul `ExternalService.java` sera à modifier, sans aucun autre impact.

De la même façon, vous pourriez aussi envisager, comme dans une architecture Hexagonale, de couper la dépendance entre le Repository et le service métier, en injectant dans ce dernier, une interface présente dans le package `spi` et dont l'implémentation serait votre Repository. Il faudrait aussi ajouter un *mapper* de l'entité métier vers l'entité à stocker, et inversement. Le service métier serait ainsi agnostique du type de stockage utilisé.  
De plus, pour vraiment isoler le service métier, vous pourriez le découpler de la technologie (ici Spring) en implémentant par exemple vos propres annotations.

Vous arrivez donc à la structuration par module suivante :

```acme
src/main/java
    ...fr.example
    ...fr.example.SpringModulithApplication.java // Spring Boot Main Application
    ...fr.example.establishment
    ...fr.example.establishment.internal // la logique métier
    ...fr.example.establishment.internal.domain
    ...fr.example.establishment.internal.domain.EstablishmentService.java
    ...fr.example.establishment.internal.entity
    ...fr.example.establishment.internal.repository
    ...fr.example.establishment.spi // les points de sortie du domaine métier
    ...fr.example.establishment.spi.dto
    ...fr.example.establishment.spi.mapper
    ...fr.example.establishment.spi.external
    ...fr.example.establishment.spi.external.ExternalService.java
    ...fr.example.establishment.spi.ExternalInterface.java
    ...fr.example.establishment.event
    ...fr.example.establishment.EstablishmentControler.java // l'API REST permettant de requêter le domain métier
    ...fr.example.establishment.EstablishmentServiceInterface.java // l'interface permettant d'accéder aux services du domaine métier
    ...fr.example.classroom
    ...fr.example.classroom.[...]
    ...fr.example.student
    ...fr.example.student.[...]
```

Jusqu'ici rien de nouveau, Spring Modulith n'intervient pas, c'est une structuration Modulaire.

### Couplage inter-modules

#### Injection de dépendance non autorisée

Spring Modulith permet de vérifier qu'aucun couplage non souhaité, mais autorisé par le compilateur Java, n'a été implémenté.  
Comment ? Grâce à un simple test unitaire !

Commencez par ajouter la dépendance `spring-modulith-starter-test` (en scope `test` bien sûr) et écrivez le test suivant :

```java
public class ModularityTest {

    @Test
    void shouldBeCompliant() {
        ApplicationModules.of(SpringModulithApplication.class).verify();
    }
}
```

Lors de son exécution, si une dépendance interdite est détectée, par exemple l'injection directe du Service `EstablishmentService.java` au lieu de l'injection via son interface `EstablishmentServiceInterface.java` dans le module *Classroom*, le test échoue et l'erreur suivante est affichée :

```log
org.springframework.modulith.core.Violations: 
- Module 'classroom' depends on non-exposed type fr.example.establishment.internal.domain.EstablishmentService within module 'establishment'!
ClassroomService declares constructor ClassroomService(ClassroomRepository, PlanningRepository, EstablishmentService) in (ClassroomService.java:0)
- Module 'classroom' depends on non-exposed type fr.example.establishment.internal.domain.EstablishmentService within module 'establishment'!
Method <fr.example.classroom.internal.domain.ClassroomService.createClassroom(fr.example.classroom.internal.entity.Classroom)> calls method <fr.example.establishment.internal.domain.EstablishmentService.getMaxNumberOfClassroomByEstablishmentId(java.util.UUID)> in (ClassroomService.java:38)
```

L'erreur est explicite, le module *Classroom* dépend du type `EstablishmentService` qui n'est pas utilisable. En injectant la dépendance au service via son interface, le problème est résolu.

#### Utilisation de type non autorisé

Le package `api` (racine du module) doit être le seul point d'entrée des autres modules.

Mais un module peut avoir besoin d'un type d'objet déclaré dans le package *spi* d'un autre module : par exemple, si ce dernier émet des évènements et que le premier a un listener sur cet évènement. Dans ce cas, le type d'évènement doit être accessible par le premier module.

Si vous exécutez de nouveau le test précédent, vous obtenez l'erreur :

```log
org.springframework.modulith.core.Violations: 
- Module 'classroom' depends on non-exposed type fr.example.establishment.spi.event.EstablishmentDeletedEvent within module 'establishment'!
EstablishmentDeletedEvent declares parameter EstablishmentDeletedEvent.onRemovedEstablishmentEvent(EstablishmentDeletedEvent) in (EstablishmentEventManager.java:0)
- Module 'classroom' depends on non-exposed type fr.example.establishment.spi.event.EstablishmentDeletedEvent within module 'establishment'!
Method <fr.example.classroom.EstablishmentEventManager.onRemovedEstablishmentEvent(fr.example.establishment.spi.event.EstablishmentDeletedEvent)> calls method <fr.example.establishment.spi.event.EstablishmentDeletedEvent.establishmentId()> in (EstablishmentEventManager.java:19)
- Module 'classroom' depends on non-exposed type fr.example.establishment.spi.event.EstablishmentDeletedEvent within module 'establishment'!
Method <fr.example.classroom.EstablishmentEventManager.onRemovedEstablishmentEvent(fr.example.establishment.spi.event.EstablishmentDeletedEvent)> has parameter of type <fr.example.establishment.spi.event.EstablishmentDeletedEvent> in (EstablishmentEventManager.java:0)
```

Spring Modulith a également une solution pour ce problème : l'ajout du fichier `package-info.java` dans le package contenant le type à exposer. Ce fichier doit associer le nom de "*l'interface nommée*" au package exposé :

```java
@org.springframework.modulith.NamedInterface("establishment-event-spi")
package fr.example.establishment.spi.event;
```

L'effet de cette déclaration est double.

* Tous les autres modules sont alors autorisés à faire référence au contenu du package `establishment.event.spi`.  
  Dans la majorité des cas, exposer tous les types d'un package n'est ni souhaité ni souhaitable. Spring Modulith vous fournit évidemment une solution pour n'exposer qu'une partie des types présents dans ce package, en utilisant les "*interfaces nommées*" via l'annotation `@NamedInterfaces` sur chacun des types à exposer :

  ```java
  @NamedInterface("establishment-deleted-event-spi")
  public record EstablishmentDeletedEvent(UUID establishmentId) implements DomainEvent {}
  ```

* Placé à la racine d'un module, le fichier `package-info.java` permet aux modules de faire référence à "*l'interface nommée*" dans des déclarations de dépendance explicites :

  ```java
    @org.springframework.modulith.ApplicationModule(allowedDependencies = "establishment::establishment-event-spi")
    package example.classroom;
  ```

  Dans ce cas, le module *Classroom* est autorisé à accéder à tous les types présents dans le package contenant l'interface nommée "*establishment-event-spi*" du module *Establishment*, mais plus aux types présents dans le package racine de ce dernier.  
  Dans le cas où le contenu du package racine et les "*interfaces nommées*" doivent être accessibles, ajouter le nom du module est une solution :

  ```java
  @org.springframework.modulith.ApplicationModule(
          allowedDependencies = {"establishment", "establishment::establishment-event-spi"}
  )
  package fr.example.classroom;
  ```

Le couplage inter-modules est géré, et avouons-le, de manière assez simple et intuitive par Spring Modulith. Qu'en est-il du couplage intra-module ?

### Couplage intra-module

Pour palier ce type de couplage, Spring Modulith n'intervient pas. Seuls les mécanismes de visibilité du compilateur Java sont utilisés.  
En effet, il n'est pas souhaitable que les services soient injectés directement. Vous allez donc rendre les implémentations non accessibles en les déclarant "*package-private*".

```java
@Service
class EstablishmentService implements EstablishmentServiceInterface {
    [...]
}

@Service
class ExternalService implements ExternalInterface {
    [...]
}
```

À ce stade, vous êtes désormais capables de développer votre application avec un couplage très faible et une structuration en packages propre.

### L'Event-Driven selon Spring Modulith

Afin de garder les modules aussi découplés que possible les uns des autres, leur principal moyen d'interaction devrait être la publication et la consommation d'événements.

Prenez par exemple la suppression d'un établissement.  
Dans ce cas, les classes, voire les étudiants, devraient également être supprimés ou mis à jour. Pour assurer un couplage très faible, cette information sera publiée par le module *Establishment* via un évènement. Ce module ne sait pas et n'a pas à savoir avec quel(s) autre(s) module(s) il partage l'information, il la diffuse et c'est aux autres modules de s'adapter.

Pour diffuser cette information, plusieurs choix s'offrent à vous :

* **La publication synchrone simplifie le modèle de cohérence**. Dans notre cas, soit l'établissement et les classes associées sont supprimés, soit aucun d'entre eux. Mais la suppression d'une classe peut également déclencher une fonctionnalité connexe non cruciale. Si celle-ci échoue, c'est toute la transaction qui échoue.
* **La consommation asynchrone évite l'expansion de la transaction originale**. Si le traitement de l'évènement échoue, ce dernier sera perdu, à moins bien entendu de mettre en œuvre un mécanisme spécifique (retry) dans chaque module à l'écoute. Dans le pire des cas, l'évènement peut même être perdu en cas de crash de l'application avant la fin de son traitement. Et vous perdez la cohérence des données entre modules.

Spring Modulith propose via la dépendance `spring-modulith-events-api` de gérer simplement les évènements, et ce, de manière asynchrone.

La création d'un évènement est réalisé via l'annotation `@DomainEvent` ou en implémentant l'interface `DomainEvent` :

```java
@NamedInterface("establishment-event-spi")
@DomainEvent
public record EstablishmentDeletedEvent(UUID establishmentId) {
}
```

```java
@NamedInterface("establishment-event-spi")
public record EstablishmentDeletedEvent(UUID establishmentId) implements DomainEvent {}
```

La publication d'un évènement de type `EstablishmentDeletedEvent` se fait ainsi :

```java
private final ApplicationEventPublisher events;

@Transactional
public void publishDeleteEstablishmentEvent(UUID establishmentId) {
  [...]
  event.publishEvent(new EstablishmentDeletedEvent(establishmentId));
}
```

Annoter la méthode du service métier (publisher) avec `@Transactional`, permet au listener de gérer la publication au moment de la validation de la transaction.

L'annotation `@ApplicationModuleListener` fournie par Spring Modulith, appliquée à la méthode en écoute (*listener* ou *consumer*), est équivalente à l'association des trois annotations suivantes :

* `@Async` permet l'exécution de la méthode de manière asynchrone.
* `@Transactional(propagation = Propagation.REQUIRES_NEW)` permet l'exécution de la méthode dans une nouvelle transaction.
* `@TransactionalEventListener` permet de déclencher l'exécution de la méthode après le commit de la transaction ayant publié l'évènement.

La consommation asynchrone d'un évènement est implémentée comme suit :

```java
private final ClassroomServiceInterface classroomServiceInterface;

@ApplicationModuleListener
void onRemovedEstablishmentEvent(EstablishmentDeletedEvent event) {
      var establishmentId = event.establishmentId();
    classroomServiceInterface.deleteByEstablishmentId(establishmentId);
}
```

Vous remarquez ici la dépendance du module *Classroom* sur l'évènement `EstablishmentDeletedEvent` présent dans le module *Establishment*. Celle-ci est mineure si on la compare avec une injection de dépendance directe du service de mise à jour des données de *Classroom* dans *Establishment*.

La gestion asynchrone d'évènements est gérée, qu'en est-il du problème de perte possible d'évènements ?  
Spring Modulith propose également un système de sauvegarde des évènements (Spring parle ici de *registre de publications*) et de rejeu automatique.  
Dans ce cas, votre publication d'évènement est réalisée :

* Soit via une base de données en utilisant la dépendance  `spring-modulith-starter-jpa` ou la dépendance `spring-modulith-starter-jdbc` (les dépendances pour MongoDB et Neo4j sont également disponibles).
  * Si vous utilisez la dépendance "jdbc", la table associée peut être créée automatiquement par Spring au démarrage de votre application. Dans ce cas, la configuration `spring.modulith.events.jdbc.schema-initialization.enabled=true` est nécessaire.  
    Sinon, vous devrez gérer vous-mêmes cette création (voir la [documentation Spring](https://docs.spring.io/spring-modulith/reference/appendix.html#schemas) à ce sujet).
  * Le rejeu des publications incomplètes peut être automatique au redémarrage de l'application en ajoutant la configuration `spring.modulith.republish-outstanding-events-on-restart=true`.
  * Le rejeu des publications incomplètes (sans date de fin en base de données) peut également être programmé. Dans l'exemple suivant, la programmation est fixée à 60 secondes après la publication initiale, avec un rejeu possible toutes les 60 secondes.

    ```java
    @Component
    @RequiredArgsConstructor
    @Slf4j
    class EventPublications {
      private final IncompleteEventPublications incompleteEvents;

      @Scheduled(fixedRate = 60_000)
      void reSubmitIncompleteEventsOlderThan() {
          log.info("Resubmit incomplete publications");
          incompleteEvents.resubmitIncompletePublicationsOlderThan(ofSeconds(60));
      }
    }
    ```

  * Il vous est également possible de filtrer les publications à republier.

  * Dans le cas d'une publication émise suite à la suppression de l'établissement ayant comme identifiant `a55c2eba-70be-4b93-b713-7ee3cdb57ac8`, la table `event_publication` utilisée par Spring Modulith pour stocker les évènements, contient les données suivantes (publication consommée par le module *Classroom* et non-consommée par le module *Student*) :

    | id                                   | listener_id                                                                                                                                   | event_type                                                        | serialized_event                                            | publication_date               | completion_date                |
    |:-------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------|:------------------------------------------------------------------|:------------------------------------------------------------|:-------------------------------|:-------------------------------|
    | e85c4255-b26d-4a77-a8c1-08d4fa3e5623 | **fr.example.student**.EstablishmentEventManager.onRemovedEstablishmentEvent(fr.example.establishment.spi.event.EstablishmentDeletedEvent)    | fr.example.establishment.spi.event.**EstablishmentDeletedEvent**  | {"establishmentId":"a55c2eba-70be-4b93-b713-7ee3cdb57ac8"}  | 2024-06-11 21:25:08.368756+00  |                                |
    | f96e1ce2-0527-4fa8-99e0-f6a899a5d8dc | **fr.example.classroom**.EstablishmentEventManager.onRemovedEstablishmentEvent(fr.example.establishment.spi.event.EstablishmentDeletedEvent)  | fr.example.establishment.spi.event.**EstablishmentDeletedEvent**  | {"establishmentId":"a55c2eba-70be-4b93-b713-7ee3cdb57ac8"}  | 2024-06-11 21:25:08.363755+00  | 2024-06-11 21:25:08.560532+00  |

* Soit via un *message broker* externe, tel que *RabbitMQ* (`spring-modulith-events-amqp`), *Kafka* (`spring-modulith-events-kafka`) ou encore *JMS* (`spring-modulith-events-jms`), si vos évènements intéressent des systèmes externes, ou par exemple si vous avez déjà un message broker en place.  
  Il vous suffit pour ça d'annoter le type d'évènements (ici, le record `EstablishmentDeletedEvent`) avec `@Externalized`.

Pour ceux souhaitant aller plus loin, Spring Modulith fournit aussi une solution basée sur les "*Moments*", via la dépendance `spring-modulith-moments`. Elle permet de capturer des moments spécifiques dans le temps et de les utiliser pour la planification.  
Imaginez qu'à chaque début de mois, chaque établissement doit envoyer les fiches de payes à ses employés par mail. Utiliser les moments pourrait dans ce cas s'avérer utile et vous pourriez imaginer l'implémenter comme suit :

```java
@Component
public class MonthlySalaryService {

    @EventListener
    public void onMonthHasPassed(MonthHasPassed event) {
      // Implémentation de la génération et de l'envoi du rapport
    }
}
```

Ici la méthode `onMonthHasPassed` sera déclenchée automatiquement par Spring Modulith à chaque changement de mois.

### Documentation

Spring Modulith fournit nativement une génération documentaire comprenant :

* les diagrammes de type C4 ou UML (au choix) de l'application et de chaque module, montrant les interactions entre eux (utilisation, dépendances, listeners),
* les "canvas" comprenant les *Services*, les *Repository*, les *Event* (publiés et/ou consommés) et les propriétés de configurations, exposés par module.

Pour générer cette documentation, ajoutez simplement ce test :

```java
    @Test
    void writeDocumentationSnippets() {
        new Documenter(ApplicationModules.of(SpringModulithApplication.class), "path/to/doc") // répertoire de génération de la documentation
                .writeModuleCanvases()
                .writeModulesAsPlantUml()
                .writeIndividualModulesAsPlantUml();
    }
```

Exemple de canvas généré pour notre module *Classroom* :

![Canvas](/assets/2024/07/01/01-canvas.png)

Exemple de diagramme C4 généré pour notre module *Classroom* :

![Canvas](/assets/2024/07/01/02-C4.png)

### Modulariser et isoler ses tests

Là encore, Spring Modulith offre des possibilités de tests très intéressantes !

* **Les tests par module :** en créant les mêmes packages au sein de vos tests que dans votre application, vous allez pouvoir tester chaque module indépendamment, en annotant simplement votre classe de tests avec `@ApplicationModuleTest`.

  ```java
  package fr.example.establishment;

  @ApplicationModuleTest
  public class EstablishmentIntegrationTests {
      // Initialisation

      @Test
      void testEstablishmentCreationAndDeletion() {
          EstablishmentDTO establishmentDTO = EstablishmentDTO.builder()
                .name("College X")
                .nbMaxClassroom(35)
                .build();

          EstablishmentDTO establishmentCreated = given().port(serverPort)
                  .contentType(APPLICATION_JSON_VALUE)
                  .body(gson.toJson(establishmentDTO))
                  .when()
                  .post("/establishments")
                  .then()
                  .statusCode(CREATED.value())
                  .extract()
                  .as(EstablishmentDTO.class);

          // Assertions
      }
  }
  ```

  À l'exécution de celui-ci, vous pouvez voir dans les traces, quel(s) module(s) est (sont) bootstrapé(s) de manière détaillée :

  ```log
    .   ____          _            __ _ _
  /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
  ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
  \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
    '  |____| .__|_| |_|_| |_\__, | / / / /
  =========|_|==============|___/=/_/_/_/
  :: Spring Boot ::                (v3.3.0)

  INFO - Bootstrapping @org.springframework.modulith.test.ApplicationModuleTest for Establishment in mode STANDALONE (class fr.example.SpringModulithApplication)…
  INFO - 
  INFO - # Establishment
  INFO - > Logical name: establishment
  INFO - > Base package: fr.example.establishment
  INFO - > Named interfaces:
  INFO -   + NamedInterface: name=<<UNNAMED>>, types=[ f.e.e.EstablishmentController, f.e.e.EstablishmentServiceInterface ]
  INFO -   + NamedInterface: name=establishment-event-spi, types=[ f.e.e.s.e.EstablishmentDeletedEvent ]
  INFO - > Direct module dependencies: none
  INFO - > Spring beans:
  INFO -   + ….EstablishmentController
  INFO -   o ….internal.domain.EstablishmentService
  INFO -   o ….internal.repository.ActivityRepository
  INFO -   o ….internal.repository.EmployeeRepository
  INFO -   o ….internal.repository.EstablishmentRepository
  INFO -   o ….spi.external.ExternalService
  INFO -   o ….spi.mapper.ActivityMapperImpl
  INFO -   o ….spi.mapper.EmployeeMapperImpl
  INFO -   o ….spi.mapper.EstablishmentMapperImpl
  ```

* Vous pouvez également démarrer votre module à tester, en incluant ceux dont il dépend, ou l'arbre complet des modules qui en dépendent, via respectivement les valeurs `DIRECT_DEPENDENCIES` et `ALL_DEPENDENCIES` du paramètre `mode` de l'annotation `@ApplicationModuleTest`.  
  La valeur par défaut du mode est `STANDALONE`.
* Vous pouvez, grâce à `Scenario`, tester vos évènements ou l'enchainement d'évènements.  
  Par exemple, si dans le cas d'une suppression d'établissement, les classes associées doivent aussi être supprimées sur réception de l'évènement correspondant :

  ```java
  @Test
  void testImpactOnClassroomInCaseOfEstablishmentDeleted(Scenario scenario) {
    // Création d'une classe liée à un établissement X dont l'id est égal à "establishmentId"
    [...]
    // Scénario d'envoi de l'évènement EstablishmentDeletedEvent
    scenario.publish(new EstablishmentDeletedEvent(establishmentId))
      .andWaitForStateChange(() -> classroomRepository.findByEstablishmentId(establishmentId))
      // on vérifie que la liste des classes associées à cet établissement est vide
      .andVerify(result -> {
          assert result.isEmpty();
      });
  }
  ```

  Autre exemple, si la suppression d'un établissement entraine la suppression d'une classe, elle-même impliquant la mise à jour des élèves (via un event reçu de *Classroom*), votre test sera :

  ```java
  @Test
  void testImpactOnStudentsInCaseOfEstablishmentDeleted(Scenario scenario) {
    // Création d'un étudiant associé à une classe
    [...]
    // Scénario d'envoi de l'évènement EstablishmentDeletedEvent
    scenario.publish(new EstablishmentDeletedEvent(establishmentId))
            // on attend la réception d'un évènement de type ClassroomDeletedEvent
            .andWaitForEventOfType(ClassroomDeletedEvent.class)
            // on vérifie
            .matchingMappedValue(ClassroomDeletedEvent::classroomId, student.getClassroomId())
            .toArrive();
  }
  ```

  Ici, vous avez utilisé la méthode `publish` mais il existe également la méthode `stimulate`, qui va vous permettre d'invoquer une méthode d'un bean :

  ```java
  @Test
  void testX(Scenario scenario) {
    // Initialisation
    [...]
    // Utilisation de stimulate()
    scenario.stimulate(() -> classroomServiceInterface.createClassroom(classroom))
          .[...]
  }
  ```

* Dernier point : en utilisant le mode Standalone au bootstrap, vous aurez certainement des problèmes dûs à l'utilisation de beans externes à votre module. Ne tombez pas dans la facilité en élargissant le scope du bootstrap, mocker plutôt ces beans (`@MockBean`), Spring Boot s'occupera de les inclure dans le contexte d'application

## Quoi d'autre ?

Comme chaque framework fourni par Spring, Spring Modulith supporte les fonctionnalités *Actuator* (`spring-modulith-actuator`) et *Observability* (`spring-modulith-observability`). Celles-ci devraient vous être très utiles pour votre monitoring.

## Convaincus ?

Spring Modulith vous permet d'avoir l'**isolation forte** de l'architecture Hexagonale.  
Celle-ci prône également l'isolation technique du domaine métier. Vous pourriez vous-aussi mettre en place ce concept, en déplaçant tout ce qui doit l'être dans les sous-packages `spi` de vos modules. Ayez simplement à l'esprit de garder du sens dans le découpage de votre application, afin de ne pas ajouter une complexité supplémentaire.

Spring Modulith vous permet également de gérer de l'**Event Driven**.  
De façon interne (base de données), il vous fournit un mécanisme de rejeu des évènements non distribués : automatique au redémarrage de l'application, ou semi-automatique via du code simple.  
Ces évènements peuvent être externalisés, via un message broker, afin d'être en mesure de partager des évènements auprès d'autres systèmes.

Tester chaque module indépendamment est également un vrai plus, tout comme son système de **génération de documentations techniques**, permettant rapidement d'identifier des dépendances non attendues.

Pour moi, [Olivier Drotbohm](https://odrotbohm.github.io/) a réalisé un travail remarquable avec ce framework. La force de **Spring Modulith** reside dans le fait que, grâce à elle, les concepts de *Separation of Concerns* et de *Bounded Contexts* peuvent être utilisés simplement et facilement, ensemble ou non, pour créer une application modulaire, faiblement couplée, testable, et évolutive.

Pour finir, n'hésitez pas à jeter un œil à notre exemple de gestion d'établissements scolaires, présent sous [GitHub](https://github.com/cedricSarre/spring-modulith-sample).  
L'application n'est certes pas complète, mais vous pourrez également constater que chaque module a son propre schéma de base de données pour une isolation des données encore plus forte, voire une meilleure anticipation d'une potentielle migration en microservices.  
Vous découvrirez enfin que nous avons ajouté un quatrième module, nommé "*core*" (souvent utilisé dans les architectures modulaires). Celui-ci contient :

* la configuration *Flyway* responsable de créer et migrer les différents schémas de base de données,
* un bean de rejeu de publications non complètement publiées, schédulé toutes les minutes
* un validateur custom sur les *Enumération* Java, utilisé au sein de DTO dans les modules,
* et enfin un `ExceptionHandler` permettant de catcher différents types d'exceptions et de retourner un "body" défini au sein de la réponse HTTP. Les exceptions qu'ils gèrent sont accessibles et donc sur-chargeables par les autres modules.

Ce module n'est pas obligatoire en soi, nous l'avons mis en place dans un souci de lisibilité et d'héritage.

## Liens Utiles

* [Externaliser ses publications dans Kafka](https://github.com/spring-projects/spring-modulith/blob/main/spring-modulith-examples/spring-modulith-example-kafka/readme.adoc)
* [Article de Mathias Verraes sur le Découplage au sein des Systèmes Distribués via les Moments](https://verraes.net/2019/05/patterns-for-decoupling-distsys-passage-of-time-event/)
