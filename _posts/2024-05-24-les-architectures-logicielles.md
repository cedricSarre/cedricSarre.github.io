---
layout: post
title: "Les Architectures Logicielles"
resume: "Le sujet de cet article est de présenter les principales architectures logicielles, avec en autre les 
architectures Monolithiques Traditionnelles, Hexagonales, Microservices, Modulaires, Orientées Évènements ou encore 
en Couches."
image: "/assets/images/softwareArchitectures.jpg"
imageCopyright: "https://fr.freepik.com/"
imageDe: Designed by vectorjuice / Freepik
readTime: "25"
excerpt: architecture logicielle software microservices hexagonale DDD modulaire
---

## L'Architecture Logicielle et ses concepts <!-- omit in toc -->

Monolithique, Client-Serveur, N-Tiers, Microservices, SOA, Hexagonale, MVC, Event-Driven... les architectures 
logicielles sont nombreuses et souvent combinées au sein d'une même application.

Mais que sont-elles vraiment, quels sont leurs principes et dans quels cas les utiliser ?  
Quels sont leurs avantages, quels risques prend-on à les utiliser ?

## Les différentes Architectures Logicielles <!-- omit in toc -->

1. [L'Architecture Monolithique Traditionnelle](#larchitecture-monolithique-traditionnelle)
2. [L'Architecture Hexagonale](#larchitecture-hexagonale)
3. [L'Architecture Microservices](#larchitecture-microservices)
4. [L'Architecture Event-Driven](#larchitecture-event-driven)
5. [L'Architecture N-Tier](#larchitecture-n-tier)
6. [L'Architecture en Couches (ou Layered)](#larchitecture-en-couches-ou-layered)
7. [L'Architecture Client-Serveur](#larchitecture-client-serveur)
8. [L'Architecture Orientée Services (SOA)](#larchitecture-orientée-services-soa)
9. [L'Architecture MVC (Modèle-Vue-Contrôleur)](#larchitecture-mvc-modèle-vue-contrôleur)
10. [L'Architecture Monolithique Modulaire](#larchitecture-monolithique-modulaire)
11. [Comparaison des différentes architectures logicielles](#comparaison-des-différentes-architectures-logicielles)

### L'Architecture Monolithique Traditionnelle

L'architecture **Monolithique** est un style d'architecture logicielle dans lequel toute l'application est conçue et 
déployée comme une seule unité.

L'application est généralement composée d'une seule base de code, où toutes les fonctionnalités sont développées, 
testées, déployées et maintenues ensemble. Toutes les parties de l'application, y compris la logique métier, 
l'accès aux données, voire la présentation utilisateur, sont habituellement intégrées dans une même base de code.

Dans une application monolithique, tous les composants sont souvent fortement couplés. Cela signifie que les 
modifications dans une partie du système peuvent avoir des effets inattendus sur d'autres parties. Ce couplage 
étroit crée une structure de code enchevêtrée, difficile à suivre et à comprendre, similaire à des spaghettis 
entremêlés.

Les nombreuses dépendances internes entre les modules et les composants peuvent être difficiles à gérer, car une 
modification dans un module peut nécessiter des changements dans plusieurs autres modules. La gestion des 
dépendances devient de plus en plus complexe à mesure que le système grandit, rendant le code encore plus confus et 
enchevêtré.

Les fonctionnalités sont rarement bien encapsulées dans des modules indépendants. Cette absence de modularité 
conduit à un code où les frontières entre les différentes responsabilités ne sont pas clairement définies, 
augmentant la complexité et la difficulté de maintenance.

À mesure que l'application monolithique grandit, il devient de plus en plus difficile de maintenir et d'ajouter de 
nouvelles fonctionnalités sans introduire de bugs ou de régressions. La complexité croissante et le manque de 
séparation claire des préoccupations rendent la maintenance coûteuse et risquée.

Cette complexité et l'enchevêtrement du code font que ce type d'architecture est connu sous le nom de "code spaghetti".

Les monolithes peuvent également poser des problèmes de scalabilité. Lorsque vous devez mettre à l'échelle 
l'application, vous devez mettre à l'échelle l'ensemble de l'application, même si seule une partie spécifique de 
celle-ci nécessite plus de ressources. Cette inefficacité peut conduire à un gaspillage de ressources et à des défis supplémentaires en matière de gestion des performances.

![](/assets/2024/05/24/02-architecture-monolithique.drawio.png)

### L'Architecture Hexagonale

L'architecture dite **Hexagonale** (ou *Domain Driven Design*) met l'accent sur la séparation des préoccupations en 
définissant clairement les interfaces externes (appelées "ports") et un utilisant des adaptateurs pour connecter ces 
interfaces aux composants internes. C'est de là qu'elle tire son autre nom "Architecture Ports et Adaptateurs".  
L'application est organisée autour d'un noyau central de logique métier (le domaine métier), qui est indépendant de 
toute infrastructure externe.  
Les ports permettent à ce noyau de communiquer avec l'extérieur sans dépendre des détails de mise en œuvre 
spécifiques.  
Les adaptateurs fournissent les mécanismes d'entrée et de sortie nécessaires pour connecter le noyau aux 
technologies externes, telles que les bases de données, les API externes, les interfaces utilisateur...  
Elle repose sur la notion de *Bounded Context* (ou "*Contexte Borné*"), qui représente un contexte métier limité et 
isolé.  
Elle favorise la testabilité en permettant le remplacement facile des composants externes par des mocks lors de 
l'exécution des tests.  
Elle offre donc une approche modulaire pour la conception d'applications, facilitant la maintenance et l'évolution 
de l'application au fil du temps.  
Elle peut autant s'appliquer aux applications monolithiques qu'aux microservices.

![](/assets/2024/05/24/03-architecture-hexagonale.drawio.png)

### L'Architecture Microservices

L'architecture **Microservices** est un style architectural dans lequel une application est construite comme un 
ensemble de petits services autonomes, chacun gérant un domaine métier spécifique.  
Chaque service est déployé indépendamment et communique généralement avec les autres services via des protocoles 
légers (HTTP/REST, gRPC, AMQP...).  
Un service est une unité autonome et indépendante de déploiement, possédant sa propre logique métier, ses données et 
sa base de code. Chaque service est habituellement conçu pour être hautement cohésif, c'est-à-dire qu'il ne traite 
qu'un seul aspect de la fonctionnalité de l'application.  
L'architecture microservices permet une évolutivité et une flexibilité élevées, car chaque service peut être 
développé, déployé et mis à l'échelle indépendamment des autres services. Cela permet aux équipes de développement 
de travailler de manière autonome sur des parties spécifiques de l'application, favorisant ainsi l'agilité et la 
réactivité aux changements.  
Mais elle introduit également une complexité opérationnelle supplémentaire, notamment en ce qui concerne la gestion 
de nombreux services et leur coordination.

![](/assets/2024/05/24/04-architecture-microservices.drawio.png)

### L'Architecture Event-Driven

L'architecture **Event-Driven** (ou *Orientée Évènements*) est un style architectural dans lequel les composants de 
l'application communiquent entre eux en réagissant aux événements qui se produisent dans le système.  
Les événements sont des messages asynchrones qui transportent des informations sur un changement d'état ou une 
action dans le système.  
Dans l'architecture événementielle, les composants réagissent aux événements en s'abonnant aux événements qui les 
intéressent et en les traitant lorsqu'ils se produisent. Les composants peuvent s'abonner à des types d'événements 
spécifiques ou être notifiés de manière proactive lorsqu'un événement se produit.  
Elle introduit cependant une complexité supplémentaire dans la gestion des événements, notamment la gestion des flux 
d'événements, la garantie de la livraison des événements ou encore la gestion des erreurs.

![](/assets/2024/05/24/05-architecture-event-driven.drawio.png)

### L'Architecture N-Tier

Également appelée "*architecture à plusieurs niveaux*", l'architecture **N-Tier** divise l'application en plusieurs 
niveaux (ou couches) distincts, chaque niveau étant responsable d'une fonctionnalité spécifique de l'application.  
Les niveaux typiques incluent la couche de présentation utilisateur, la logique métier et la persistance des données
(accès aux données).

L'architecture N-Tier permet une scalabilité horizontale si les N couches sont séparées ; par exemple, une couche 
présentation sur un serveur Web, une couche métier sur un serveur d'application et la couche base de données. La 
scalabilité ici est réalisée en ajoutant des instances de serveur supplémentaires pour gérer une charge croissante 
sur chaque niveau de l'application.

![](/assets/2024/05/24/06-architecture-n-tier.drawio.png)

### L'Architecture en Couches (ou Layered)

Assez similaire à l'architecture N-Tier, l'Architecture **en Couches** se distingue par sa séparation en couches 
logiques, contrairement à la séparation en couches en "tier" physiques ou logiques du N-Tier. De plus, la 
communication entre les couches est ici verticale (et non horizontale comme son homologue) et donc sa scalabilité 
est limitée contrairement au N-Tier où chaque couche peut être "scalée" indépendamment.

![](/assets/2024/05/24/07-architecture-couches.drawio.png)

### L'Architecture Client-Serveur

L'Architecture **Client-Serveur** est un modèle d'architecture dans lequel les fonctionnalités de l'application sont 
réparties entre deux types de composants : les clients et les serveurs.  
Les clients sont des applications qui envoient des requêtes à des serveurs pour obtenir des données ou exécuter des 
opérations.  
Les serveurs sont des systèmes qui répondent aux requêtes des clients en fournissant des données ou en exécutant des
traitements.

L'Architecture Client-Serveur favorise généralement la séparation des préoccupations entre la présentation 
utilisateur (côté client) et la logique métier et les données (côté serveur).  
Cela permet une meilleure gestion des responsabilités et une évolutivité plus facile de l'application.

Les architectures Client-Serveur peuvent être conçues pour être évolutives en ajoutant des serveurs supplémentaires 
pour gérer une charge croissante de requêtes de clients.

Ce type d'architecture est beaucoup moins utilisé de nos jours, même si certaines applications de bureau (ERP par 
exemple), ou encore par les jeux en réseau où le serveur gère la logique du jeu et où les joueurs sont les clients, 
se basent sur du Client-Serveur.

![](/assets/2024/05/24/08-architecture-client-serveur.drawio.png)

### L'Architecture Orientée Services (SOA)

L'Architecture **Orientée Services** est un style architectural qui organise une application comme un ensemble de 
services interconnectés et indépendants.

Ce type d'architecture ressemble fortement aux microservices ? C'est vrai, mais avec quelques différences importantes.
L'architecture SOA est plutôt une architecture à l'échelle d'une entreprise. Autre différence importante, dans une 
architecture SOA, les services communiquent à travers un ESB.

Quatre composants la structure :

* les services ou composants logiciels autonomes offrant une fonctionnalité spécifique à travers des interfaces 
  standardisées. Chaque service est généralement conçu pour réaliser une tâche ou une fonction métier spécifique, et 
  peut être déployé et géré de manière indépendante.
* le registre de services dans lequel ces derniers sont enregistrés et donc découverts par d'autres (peut faire 
  penser à une solution de Service Discovery)
* le Bus de services (ou ESB) responsable de gérer la communication entre services et leur orchestration
* le contrat de service définissant son rôle et comment y accéder

![](/assets/2024/05/24/09-architecture-soa.drawio.png)

### L'Architecture MVC (Modèle-Vue-Contrôleur)

L'architecture **MVC** est utilisée pour le développement d'applications web. Elle divise une application en trois 
composants principaux : le Modèle (POJO, Entités...), la Vue (JSP, Thymeleaf...) et le Contrôleur (Servlets, Spring 
MVC Controllers...).

La Vue envoie les données saisies par l'utilisateur au Contrôleur qui les traite, applique les règles métiers et met 
à jour le Modèle, qui lui-même met à jour la base de données. Puis le Contrôleur sélectionne la vue appropriée pour 
présenter les données à l'utilisateur.

Chaque composant a donc une responsabilité différente, facilitant ainsi la maintenance, la testabilité et les 
impacts potentiels de modification d'un composant sur les autres.

Le framework Spring MVC est un parfait exemple d'implémentation de ce type d'architecture.

![](/assets/2024/05/24/10-architecture-mvc.drawio.png)

### L'Architecture Monolithique Modulaire

L'architecture **Monolithique Modulaire** combine les avantages de l'architecture monolithique avec ceux du Domain 
Driven Design (voire de l'Event-Driven), offrant ainsi une solution bien équilibrée pour de nombreux types 
d'applications.

Contrairement aux architectures distribuées comme les microservices ou SOA, elle simplifie le déploiement, la 
gestion et la maintenance de l'application en ne nécessitant qu'un seul environnement d'exécution.  
En divisant l'application en modules distincts définis par leur *Bounded Context*, chaque module peut être développé,
testé et maintenu de manière indépendante, ce qui favorise la ré-utilisabilité du code et la séparation des 
préoccupations.

En évitant les communications réseau et en partageant les ressources au sein du même processus d'exécution, elle 
peut offrir des performances supérieures à celles d'une architecture distribuée.

Enfin, un module des modules peut très facilement être extrait pour fonctionner de manière autonome (pour des 
raisons de charge ou de performances) dû au faible couplage de cette architecture.

Dans ce cas, quelles sont les solutions qui permettraient aux différents modules de récupérer les données 
appartenant à un autre module, sachant que celles-ci doivent avoir un couplage faible, voire très faible, une mise 
en œuvre simple et des données mises à jour en temps réel ou très proche du temps réel ?

![](/assets/2024/05/24/01-couplage-complexité-synchro.png)

Généralement, nous privilégions **la messagerie** pour un plus haut niveau d'autonomie, et si une légère 
désynchronisation des données entre les modules est acceptable.  
**Le partage des données à travers une base de données** est privilégié si les performances, la fiabilité et 
l'évolutivité sont primordiales.  
**Le transfert de fichiers** est utilisé si le temps de synchronisation des données n'est pas un enjeu majeur.  
Enfin, **l'appel direct** (dépendance) est la solution retenue pour assurer une forte cohérence et lorsque le niveau 
maximal d'autonomie des modules n'est pas requis ; ou lorsque la cohérence des données entre modules est essentielle.

Il est évidemment possible de mélanger ces différents types de solutions.  
Lorsqu'il s'agit d'une architecture modulaire, il apparait alors préférable d'utiliser conjointement l'appel direct 
et la messagerie permettant ainsi à certains modules de communiquer de manière synchrone et à d'autres de manière 
asynchrone.

Si nous devions résumer, l'architecture Monolithique Modulaire offre un compromis élégant entre la simplicité de 
l'architecture monolithique, le découplage du Domain Driven Design et la modularité des architectures distribuées.

![](/assets/2024/05/24/11-architecture-monolithique-modulaire.drawio.png)

Mais cette architecture peut également poser des problèmes de scalabilité. Tout comme l'architecture monolithique, 
lorsque vous devez mettre à l'échelle l'application, vous devez mettre à l'échelle l'ensemble de 
l'application, même si seule une partie spécifique de celle-ci nécessite plus de ressources. Mais grâce à son 
couplage très faible (ou inexistant), les modules sont autonomes et peuvent alors être extrait en tant que services 
(ou microservices) externes, rendant ainsi la scalabilité aisée.

Ce type d'architecture est fait pour être mis en place lors du "build" d'une application sans savoir si une architecture 
microservices sera nécessaire dans le futur.  
Elle est également une bonne solution lorsqu'on souhaite commencer une migration d'un monolithe vers des 
microservices. Il "suffit" dans un premier temps de restructurer son application sous forme de modules les plus 
indépendants possibles, en gardant la même base de code et en gardant les mêmes tests d'intégration. Puis, vient le 
temps de migrer ses tests d'intégration par module et enfin, de sortir petit à petit les modules en tant que 
microservices.

### Comparaison des différentes architectures logicielles

Que ce soit en architecturant une application ou en la développant, nous avons toujours des critères à respecter 
parmi la Maintenance, la Testabilité, la Complexité, la Flexibilité, la Scalabilité, les Performances et la Facilité 
de Déploiement.

Il est très compliqué de tous les réunir et souvent non nécessaire. Mais alors quel type d'architecture choisir en 
fonction de vos critères d'acceptance ? Ce tableau récapitulatif pourrait vous y aider.

|        **Architectures**        | **Maintenance** | **Tests** | **Complexité** | **Flexibilité** | **Scalabilité** | **Performances** | **Déploiement** |
|:-------------------------------:|:---------------:|:---------:|:--------------:|:---------------:|:---------------:|:----------------:|:---------------:|
|        **Monolithique**         |        -        |     -     |       -        |        +        |        -        |        -         |        +        |
|   **Monolithique Modulaire**    |        +        |     +     |       -        |        +        |       -/+       |       -/+        |        +        |
|       **Client-Serveur**        |       -/+       |    -/+    |      -/+       |       -/+       |        -        |       -/+        |        -        |
|           **N-Tier**            |        -        |    -/+    |      -/+       |       -/+       |        +        |       -/+        |       -/+       |
|         **En Couches**          |       -/+       |    -/+    |      -/+       |       -/+       |       -/+       |       -/+        |       -/+       |
|        **Microservices**        |        -        |    -/+    |       -        |        +        |        +        |        +         |        -        |
|             **SOA**             |        -        |     -     |       -        |        +        |        +        |        +         |        -        |
|     **Orientée Événements**     |        -        |     +     |       -        |        +        |        +        |        +         |        -        |
|         **Hexagonale**          |        +        |     +     |      -/+       |        +        |       -/+       |       -/+        |        +        |
| **MVC (Modèle-Vue-Contrôleur)** |       -/+       |    -/+    |      -/+       |       -/+       |       -/+       |       -/+        |       -/+       |
