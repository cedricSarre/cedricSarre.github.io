---
layout: post
title: "Les Différents Patterns de Transactions Distribuées"
description: "Le sujet de cet article est de présenter les différents Patterns de Transaction Distribuées, tels que 
2PC/3PC, SAGA, TCC, Transactional Outbox ou encore Eventual Consistency.
Lequel sera le plus adapté à votre besoin ? Lequel avez-vous mis en place sans forcément vous rendre compte de ses 
inconvénients ?"
resume: "Le sujet de cet article est de présenter les différents Patterns de Transaction Distribuées, tels que 
2PC/3PC, SAGA, TCC, Transactional Outbox ou encore Eventual Consistency.
Lequel sera le plus adapté à votre besoin ? Lequel avez-vous mis en place sans forcément vous rendre compte de ses 
inconvénients ?"
image: "/assets/images/distributed-transactions.png"
readTime: "25"
excerpt: patterns transactions distribuées distributes 2PC 3PC SAGA Outbox Consistency ACID Blockchain
---

## Table des matières <!-- omit in toc -->

1. [Introduction](#introduction)
2. [Définitions et Rappels](#définitions-et-rappels)
   1. [Les Transactions](#les-transactions)
   2. [Les Transactions Distribuées](#les-transactions-distribuées)
3. [Les Patterns de Transactions Distribuées](#les-patterns-de-transactions-distribuées)
   1. [2PC (Two-Phase Commit)](#2pc-two-phase-commit)
   2. [3PC (Three-Phase Commit)](#3pc-three-phase-commit)
   3. [SAGA](#saga)
      1. [Saga Orchestration](#saga-orchestration)
      2. [Saga Choreography](#saga-choreography)
   4. [TCC (Try-Confirm/Cancel)](#tcc-try-confirmcancel)
4. [Patterns de fiabilité et de cohérence](#patterns-de-fiabilité-et-de-cohérence)
   1. [Transactional Outbox](#transactional-outbox)
   2. [Eventual Consistency](#eventual-consistency)
5. [Et la Blockchain, on en parle ?](#et-la-blockchain-on-en-parle-)
6. [Conclusion](#conclusion)
7. [Liens Utiles](#-liens-utiles)

## Introduction

Depuis quelques années, les architectures microservices et les systèmes distribués se multiplient, devenant une norme. Mais chaque solution, parfaite sur le papier, apporte son lot de difficultés et de complexité, comme par exemple la gestion des transactions distribuées, c'est-à-dire à travers de multiples services et bases de données.  

Les microservices, c'est top me direz-vous ! Les applications sont isolées, autonomes et gèrent leurs propres données... Ok, mais si chaque service fonctionne de manière autonome, avec ses propres ressources et bases de données, ayez bien à l'esprit qu'il est généralement essentiel que les données et les opérations restent cohérentes à travers l'ensemble du système. La gestion de la cohérence des données, surtout lorsqu'elles traversent plusieurs briques logicielles, entraîne de vrais défis, en particulier en termes de **cohérence**, de **fiabilité** et bien évidemment de **résilience**.

C'est là qu'interviennent les **patterns de transactions distribuées**. Ceux-ci offrent des solutions élégantes pour gérer les transactions et les erreurs dans des systèmes où plusieurs services ou bases de données sont impliqués. Parmi les plus populaires, on trouve *Two-Phase Commit* (2PC), *Three-Phase Commit* (3PC), *Saga*, et d'autres approches permettant de garantir la consistance éventuelle, tout en assurant une gestion fine des erreurs et des "compensations" (rollback ou presque...).

Dans cet article, j'explorerai en détail ces différents patterns, en mettant en lumière leurs avantages, les défis que représente leur mise en œuvre et des exemples d'applications concrètes. Nous verrons également comment ces approches s'intègrent dans des systèmes modernes utilisant des message brokers et des architectures asynchrones pour améliorer la résilience et la scalabilité des applications distribuées.

## Définitions et Rappels

Avant d'aborder les différents patterns, (re)voyons ensemble les différentes notions liées aux transactions.

### Les Transactions

Une **transaction** est *une séquence d'opérations traitées comme une unité indivisible. Elle garantit que les opérations qu’elle englobe sont soit entièrement exécutées, soit entièrement annulées. Si une transaction échoue à n'importe quel moment, toutes ses opérations doivent être annulées, afin de laisser le système dans un état cohérent.*

Le modèle **ACID**, ça vous parle ? C'est tout à fait ça. Une transaction doit répondre à 4 règles clés :

* **A**tomicité : *Soit toutes les opérations de la transaction sont exécutées avec succès, soit aucune d’entre elles n’est exécutée.*  
Cette propriété garantit que la transaction est une unité dite indivisible.
* **C**ohérence : *la base de données passe d’un état cohérent à un autre état cohérent après une transaction*.  
Pour simplifier, une transaction transforme les données de manière à respecter les règles et contraintes définies sur la base de données (clés primaires, contraintes d'intégrité, d'unicité...).
* **I**solation : *Les transactions doivent être exécutées de manière isolée les unes des autres.*  
Les opérations d'une transaction ne doivent pas être visibles aux autres transactions tant qu’elles ne sont pas terminées.
* **D**urabilité : dès qu’une transaction est validée (*commit*), les changements apportés par celle-ci doivent être permanents, même en cas de défaillance du système.

Sur certaines bases de données SQL, la définition de l'isolation n'est pas forcément vraie, tout dépend du niveau d'isolation des transactions que vous avez choisi parmi les 4 généralement disponibles :

* **Read UnCommitted** ne respecte pas complètement l'isolation
* **Read Committed** respecte partiellement l'isolation
* **Repeatable Read** respecte quasi-totalement l'isolation
* **Serializable** respecte complètement l'isolation

> Pour en savoir plus, je vous invite à lire cette [page](https://en.wikipedia.org/wiki/Isolation_(database_systems)) issue de Wikipédia.

### Les Transactions Distribuées

Une transaction distribuée implique plusieurs systèmes ou bases de données répartis sur plusieurs nœuds, serveurs ou même sites géographiques. Contrairement à une transaction classique, qui concerne une seule base de données, une transaction distribuée *doit garantir la cohérence et l'intégrité des données à travers plusieurs systèmes indépendants, voire hétérogènes*.

Dans un contexte distribué, *une transaction doit être capable de s’étendre sur plusieurs ressources, et chaque ressource doit participer à l’opération transactionnelle*. Par exemple, un système pourrait impliquer plusieurs bases de données ou services web qui doivent collaborer dans le cadre d'une seule transaction. L'objectif reste de maintenir les propriétés **ACID** dans un environnement réparti, ce qui est clairement plus complexe que dans un environnement monolithique.

Les transactions distribuées présentent **plusieurs défis de taille** par rapport aux transactions classiques :

* La **Coordination** : coordonner différentes bases de données et services est complexe. Par exemple, un service peut réussir à effectuer une opération alors qu'un autre service échoue. Comment gérer ce genre de cas ?
* La **Fiabilité et la Résilience** : les pannes peuvent se produire à tout moment dans un système distribué. Cela nécessite donc des mécanismes robustes pour gérer l’intégrité de la transaction.
* La **Latence** : les communications entre différents systèmes répartis peuvent entraîner des latences importantes. Les transactions doivent être conçues de manière à minimiser les délais trop longs.

> **A noter** : les bases de données NoSQL, souvent conçues pour des systèmes distribués, sacrifient certaines propriétés ACID pour garantir des performances élevées et la haute-disponibilité.

L'exemple auquel tout le monde pense assez logiquement en lisant ces objectifs, est une application de *e-commerce* ayant par exemple 4 services : la commande, la gestion des stocks, le paiement et la livraison.  
Chacun de ces services peut être sur un serveur différent et utiliser une base de données différente.  
La transaction doit garantir que si l’un de ces services échoue (par exemple, le paiement échoue), toute l’opération - réservation du stock et enregistrement de la commande - doit être annulée, assurant ainsi une cohérence globale.  
Il est évident que dans un tel système, la notion d'**Atomicité** au sens propre du terme est perdue. Ici on parle alors de **cohérence à terme** par opposition à la **cohérence immédiate**.

Maintenant, je suppose que vous vous rendez compte de la difficulté de tels systèmes distribués. Comment les rendre coordonnés, fiables, résilients et rapides ? Pour ça, il existe plusieurs patterns de coordination, tous assez différents, tous ayant leurs avantages et leurs inconvénients. Ouf ! 😮‍💨

## Les Patterns de Transactions Distribuées

Gardez en tête l'exemple d'un système applicatif d'e-commerce, c'est celui-ci qui servira à illustrer les différents patterns.

### 2PC (Two-Phase Commit)

Ce pattern est utilisé pour garantir l'atomicité des transactions distribuées en deux phases.

* **Phase de préparation** :  
  * Le coordinateur demande à tous les participants s'ils peuvent valider la transaction.  
  * Chaque participant répond avec un vote (prêt ou échec).  
  * Si un participant répond "échec" lors de la phase de préparation, le coordinateur envoie une commande d'annulation à tous les participants qui ont répondu "prêt".
  * Si tous les participants sont prêts, le coordinateur passe à la phase suivante.
* **Phase de validation** :  
  * Si tous les participants répondent qu'ils sont prêts, le coordinateur envoie une commande de validation à tous les participants.  
  * Si un participant vote pour l'échec, le coordinateur envoie une commande d'annulation.

"*Mais qui est ce coordinateur ?*" me direz-vous, et vous avez raison, je n'en ai pas encore parlé. Le coordinateur va dépendre du pattern de transactions distribuées utilisé.  
Pour **2PC**, il s'agira généralement d'un module/composant indépendant afin d'assurer la séparation des préoccupations (*separation of concerns*) au sein du système.  
"*Quel est son rôle ?*", simplement de maintenir le cycle de vie de la transaction, en appelant les différents modules concernés lors des deux phases. Si, à un moment donné, un module ne se prépare pas, ou si un module ne valide pas la transaction, le coordinateur est en charge d'interrompre la transaction et il commence alors la phase de compensation (*rollback*). Il est plus communément appelé "*Transaction Manager*".

Le schéma ci-dessous représente les différents use-cases de 2PC dans un système e-commerce.

![Ecommerce 2PC](/assets/2025/04/01/2PC.svg)

**Avantages** :

* 2PC est un protocole de cohérence très fort.  
Tout d'abord, les phases de préparation et de validation garantissent que la transaction est atomique. La transaction se terminera soit par un retour réussi de tous les modules, soit par une absence de modification de tous les modules.
* 2PC permet l'isolation lecture-écriture.  
Cela signifie que les modifications apportées à une entité ne sont pas visibles tant que le coordinateur ne les a pas validées.

**Inconvénients** :

* Bien que 2PC résolve le problème d'atomicité, il n'est pas vraiment recommandé pour de nombreux systèmes parce qu'il est synchrone, et donc bloquant.  
Le protocole doit verrouiller l'objet qui sera modifié avant que la transaction ne se termine.  
Si un objet "préparé" venait à être modifié, la phase de validation pourrait ne pas fonctionner.
* Les pannes réseau, ou d'un participant, peuvent entraîner des verrouillages prolongés.

Pour résumer, 2PC est un choix cohérent pour les systèmes nécessitant une **cohérence stricte**, il respecte le principe d'isolation des transactions, mais il n'est pas adapté aux systèmes à grande échelle ou à forte disponibilité, où les performances sont critiques.

### 3PC (Three-Phase Commit)

C'est une extension du 2PC avec une phase supplémentaire pour réduire les risques de blocage.

* **Phase de canCommit** :
  * Le coordinateur demande si les participants peuvent préparer la transaction.
  * Si un participant répond "échec" à cette étape, la transaction est annulée immédiatement.
* **Phase de preCommit** :
  * Si tous les participants répondent "oui", le coordinateur envoie une pré-commande de validation.
  * Si un participant ou le coordinateur échoue après cette phase, les participants passent en état de "*préparation à annuler*" ou à "*valider*" en fonction du dernier message reçu.
  * Si une erreur survient, une commande d'annulation est envoyée.
* **Phase de doCommit** :
  * Le coordinateur envoie la commande finale de validation.
  * Si une erreur se produit ici, une commande d'annulation est envoyée.

Tout comme pour 2PC, le coordinateur ici est un module/composant indépendant, appelé "Transaction Manager".

![Ecommerce 3PC](/assets/2025/04/01/3PC.svg)

**Avantages** :

* Contrairement à 2PC, 3PC introduit une phase supplémentaire (*CanCommit*), qui permet aux participants de confirmer s’ils peuvent effectuer une transaction avant de passer à la phase de préparation.
* En cas de panne réseau ou de défaillance du coordinateur, les participants peuvent prendre une décision autonome basée sur l'état de la transaction.
* 3PC est conçu pour minimiser les risques de blocage total du système, le rendant plus tolérant aux pannes dans des environnements distribués.
* Contrairement à 2PC, les participants peuvent revenir à un état cohérent sans attendre indéfiniment une réponse du coordinateur.

**Inconvénients** :

* 3PC est plus complexe que 2PC et il implique plus d'étapes de communication, introduisant donc plus de latence. Il est donc moins, voire peu, performant.

Pour résumer, en raison de sa complexité et de ses exigences, 3PC est rarement utilisé en pratique. Les systèmes modernes préfèrent des alternatives comme **SAGA** ou des approches basées sur la **cohérence éventuelle**.

> **A noter** : que ce soit 2PC ou 3PC, l'utilisation d'un message broker est inutile car les échanges sont gérés via les protocoles réseau (HTTP, TCP/IP...).

### SAGA

Le pattern **Saga** est sans doute le plus connu de tous. Il divise une transaction longue en une série de transactions plus petites, chacune avec une action compensatoire pour annuler l'effet en cas d'échec.  

* Chaque étape est une transaction locale.
* Si une étape échoue, une série de transactions compensatoires est exécutée pour annuler les étapes précédentes (*rollback*).

Il y a deux manières de mettre en œuvre le pattern *Saga* : par **Choreography** ou par **Orchestration**. Ces deux approches définissent comment les différents services s’organisent pour gérer "la Saga".

#### Saga Orchestration

Dans le modèle **Orchestration**, un coordinateur central (ou *orchestrateur*) prend en charge la gestion de "la saga".  
Ce dernier appelle les services dans un ordre précis et attend les résultats de chaque étape avant de passer à la suivante.  
Si une étape échoue, le coordinateur décide de lancer les actions de compensation pour les étapes précédentes.  

Dans notre exemple d'e-commerce, l'orchestrateur gère la réservation, le paiement, et la livraison en appelant les services dans un ordre précis, en attendant les réponses de chaque service et en lançant des compensations si nécessaire.

![ECommerce Saga et Orchestration](/assets/2025/04/01/Saga_Orchestration.svg)

**Avantages** :

* Conçu pour les environnements microservices où chaque service possède son propre domaine de responsabilité et sa propre base de données.
* La présence d'un orchestrateur centralisé simplifie la coordination inter-services.
* Contraiement à 2PC, "Saga Orchestration" est **non bloquant** car chaque étape de la Saga est indépendante.
* Il est capable de gérer les échecs partiels. L'orchestrateur peut appliquer des actions compensatoires en cas d'échec d'une étape, permettant de maintenir une **cohérence éventuelle**.
* L'orchestrateur a une vue complète de l'état de la saga, ce qui facilite la supervision et la détection des échecs.

**Inconvénients** :

* **Complexité accrue** dans la gestion des transactions compensatoires.  
Écrire une logique de compensation pour chaque étape n’est pas trivial. Contrairement à un simple "*Ctrl+Z*" qui annule la dernière action de manière isolée, les transactions distribuées impliquent plusieurs services interconnectés, chacun ayant ses propres contraintes et effets secondaires. Il faut donc anticiper des cas comme l’annulation d’un paiement après l’expédition d’un produit, ou encore gérer des erreurs de compensation pour éviter des boucles infinies. 🤯  
Des frameworks comme *Temporal.io*, *Camunda*, *Axon Framework* ou encore *Spring Cloud Data Flow* peuvent vous aider car ils permettent d'orchestrer des workflows transactionnels, tout en ayant un mécanisme de compensation.  
* Non adapté pour toutes les opérations nécessitant une stricte atomicité. Ici, on parle de **cohérence éventuelle**.
* Si l'orchestrateur tombe en panne, le système entier peut être impacté. Sa résilience doit être au coeur des préoccupations lors de sa conception.
* Une **latence accrue**, l'orchestrateur introduisant une couche supplémentaire de communications et de traitements.

Pour résumer, le modèle **Orchestration** du pattern Saga est pensé pour une architecture Microservices, mais la cohérence reste **éventuelle** (non applicable au domaine bancaire par exemple) et l'orchestrateur, étant perçu comme un **SPOF** (*Single Point Of Failure*), doit être résilient.

#### Saga Choreography

Contrairement au modèle *Orchestration*, il n'y a pas de coordinateur central. Chaque service qui participe, connaît les autres services et réagit de manière autonome aux événements déclenchés par ces derniers.

Les services échangent des événements ou des messages et chacun est responsable de sa propre logique de compensation en cas d’échec.

Dans notre système e-commerce, chaque service réagit à des événements comme par exemple "*Paiement confirmé*" ou "*Réservation annulée*", sans qu'il y ait pour autant un coordinateur central pour orchestrer l'ensemble.

Avec ce modèle, un *message broker* (Kafka, RabbitMQ...) est souvent utilisé, voire recommandé, pour publier et souscrire aux événements de chaque étape. Mais, fonction de vos appétences, ajouter un message broker à votre système est soit un avantage, soit un inconvénient 😊

Dans le schéma suivant, je n'ai pas illustré le message broker afin d'en simplifier sa compéhension, vous devez juste imaginer que chaque échange passe par celui-ci.

![ECommerce Saga et Choreography](/assets/2025/04/01/Saga_Choreography.svg)

**Avantages** :

* Contrairement à la Saga de type Orchestration, chaque service est **responsable de son rôle**, réduisant la dépendance à un orchestrateur centralisé et supprimant ainsi le principal *SPOF* du système applicatif.
* Chaque service publie des événements et réagit aux événements d'autres services, ce qui favorise la **scalabilité** et l'**indépendance** des composants du système (voire des équipes).
* La méthode Choregraphy facilite l'ajout de nouveaux services sans avoir à modifier un orchestrateur centralisé.
* Les services ne dépendent que des événements qu'ils consomment ou émettent, ce qui permet une **flexibilité accrue** de leur évolution.

**Inconvénients** :

* L'absence d'une vue centralisée rend **difficile la supervision** et donc la détection d'échecs, en particulier dans des systèmes complexes.
* Les services doivent connaître le **format des événements** et leur signification .  
  > 💡**Tips** : faites en sorte que les modifications des contrats d'interfaces soient rétrocompatibles.

Au final, la méthode **Choregraphy** permet un couplage plus faible qu'avec *Orchestration*.  
Elle est plus adaptée à une application où de nouveaux services peuvent apparaitre et disparaitre.  
Toutefois, veillez à bien penser les étapes compensatoires de chaque module afin d'éviter les échecs en cascade.

Vous l'aurez compris, quel que soit le modèle choisi, le pattern **Saga** se distingue des patterns classiques comme 2PC par sa capacité à garantir une **cohérence éventuelle** (donc non ACID) tout en évitant les blocages de ressources.  

Chaque étape d'une transaction est indépendante et je vous invite à l'accompagner d'une action compensatoire en cas d'échec.

### TCC (Try-Confirm/Cancel)

**TCC** est un pattern pour les transactions distribuées qui consiste en trois étapes :

* **Try** : réserve les ressources nécessaires.  
Si une réservation échoue, la transaction est annulée immédiatement.
* **Confirm** : Une fois que les ressources ont été réservées avec succès, la deuxième étape consiste à finaliser la transaction. À ce stade, la transaction est considérée comme terminée et les modifications sont appliquées de manière permanente.  
Si une erreur survient après cette étape, la transaction ne peut pas être annulée, car elle est déjà validée.
* **Cancel** : Si une erreur se produit après la réservation des ressources, mais avant leur confirmation, une commande d'annulation est envoyée pour libérer les ressources réservées.  
Cela permet de revenir en arrière de manière sûre et de rétablir l'état du système avant la tentative de transaction.

![ECommerce TTC](/assets/2025/04/01/TTC.svg)

**Avantages** :

* **Flexibilité accrue** : la phase de réservation permet de réserver les ressources de manière temporaire et non bloquante, et seules les étapes de confirmation ou d'annulation prennent effet de manière définitive.  
Il est donc plus flexible que le 2PC qui utilise un verrouillage strict des ressources, et ce, pendant l'ensemble du processus.
* **Isolation des étapes** : TCC garantit que les étapes de la transaction sont bien isolées, ce qui permet une gestion plus fiable des ressources dans un environnement distribué.
* **Compensation en cas d'échec** : TCC offre une compensation en cas d'échec (via l'annulation). Si quelque chose échoue après la phase de réservation, les ressources peuvent être libérées rapidement, ce qui minimise l'impact sur le système.
* **Plus flexible que 2PC** : chaque service gère indépendamment sa phase de validation ou d'annulation, alors que 2PC impose un verrouillage global et un coordinateur pour la gestion de la transaction.

**Inconvénients** :

* **Gestion complexe des compensations** : la gestion des réservations et des annulations peut devenir complexe car chaque service doit être capable de gérer les compensations de manière fiable et cohérente.
* **Pas d'atomicité globale** : bien que chaque étape soit atomique au sein d'un service donné, les différentes étapes de la transaction peuvent échouer de manière indépendante, nécessitant une gestion des compensations pour maintenir la cohérence des données à travers les services.
* **Complexité accrue** :
  * un grand nombre de services doit être capable de gérer l'état de la transaction et de communiquer efficacement pour garantir que les annulations et confirmations se produisent au bon moment.
  * des scénarios complexes, tels que des échecs simultanés dans différents services ou des situations où la gestion des compensations devient impossible, peuvent entraîner des états incohérents.

En fin de compte, le pattern **TCC** est une solution efficace pour gérer les transactions distribuées.  
Il permet de réserver les ressources de manière temporaire, puis de confirmer ou annuler la transaction selon les résultats. Ce pattern est plus flexible que le 2PC car il n'impose pas un verrouillage global des ressources.  
Cependant, sa mise en œuvre peut être complexe, surtout quand il y a beaucoup de services impliqués, car chacun d'eux doit gérer l'annulation ou la confirmation de manière fiable.

## Patterns de fiabilité et de cohérence

Nous avons vu les patterns de transactions distribuées **2PC**, **3PC**, **SAGA** et **TCC**.  
Ces derniers privilégient la gestion de l'**atomicité** et de la **cohérence des transactions** réparties sur plusieurs services et bases de données.  
Ils garantissent ainsi que l'ensemble de la transaction soit entièrement complété avec succès, ou complètement annulé en cas d'échec.

Cependant, il existe d'autres types de patterns qui ne visent pas une atomicité globale. N'ayant trouvé aucun terme pour qualifier cette catégorie de patterns, je les qualifierai de "**Patterns de fiabilité et cohérence**".  
Ces approches se concentrent principalement sur la gestion des incohérences temporaires et la fiabilité des systèmes distribués. Les deux principaux patterns de cette catégorie sont **Transactional Outbox** et **Eventual Consistency**.

### Transactional Outbox

Méconnu, le pattern **Transactional Outbox** (que nous pourrions traduire par "*Boite aux lettres transactionnelle*") garantit que les mises à jour dans une base de données et les messages envoyés à un messages broker se produisent de manière atomique. On parle alors **d'atomicité locale**. La base de données garantit ainsi la cohérence transactionelle et le message broker gère la distribution asynchrone.

Dans un système d'e-commerce :

* Le service de commande crée une commande en base de données et écrit un message dans l'*outbox* (table en base de données), et ce **dans la même transaction**.
  * Si une transaction échoue avant de mettre à jour la base de données et l'écriture dans l'outbox, la transaction entière est annulée, y compris l'écriture de l'outbox.
* Un service dit "*Message Relay*" scanne régulièrement la table *outbox* pour y trouver les messages non encore envoyés.
* Ces messages sont ensuite publiés dans un messages broker pour diffusion aux services consommateurs (stock, paiement, livraison).
  * Si la transaction réussit mais l'envoi des messages échoue, les messages restent dans l'outbox jusqu'à ce qu'ils soient ré-envoyés avec succès.
* Une fois un message envoyé avec succès, il est marqué comme traité ou supprimé de l’outbox.

J'entends déjà votre question "*Pourquoi ne pas écrire directement dans le message broker plutôt que l'outbox ?*" 😁  

Si la sauvegarde de la commande en base de données réussit, mais que l'écriture dans le message broker échoue, le message est perdu. Il vous faudrait dans ce cas implémenter un mécanisme de retry ou un mécanisme de rollback de la commande, et vous en conviendrez, c'est moins facile.  
De plus, on pourrait également imaginer le cas où l'écriture dans le message broker réussit, mais la sauvegarde de la commande échoue. Dans ce cas, le système se retrouverait avec un message diffusé pour une opération inexistante, et potentiellement, les services de paiement, de réservation de stock et de planification de livraison auraient réalisé des actions à tort... 😥

Comme précédemment, je n'ai pas mentionné de message broker dans le schéma suivant afin d'en simplifier sa compéhension.

![ECommerce Transactional Outbox](/assets/2025/04/01/TransactionalOutbox.svg)

**Avantages** :

* La **simplicité** dans la mise en œuvre.
* La **garantie** que les messages sont envoyés grâce à la persistance dans l’outbox.
* La **cohérence stricte** de l'opération principale.  
Dans notre cas, la création d'une commande et son insertion dans l'outbox sont réalisées de manière **atomique**.
* **Non bloquant**. Contrairement à 2PC, où l'ensemble des participants attendent la validation ou l'annulation, ici les opérations asynchrones évitent tout blocage.
* **Scalable**.

**Inconvénients** :

* Nécessite un **Message Relay** pour lire l'outbox.
* La table *outbox* peut devenir un point de **contention** si le Message Relay est indisponible sur une longue période ou si les services consommateurs sont lents à traiter les messages.

En définitive, le pattern **Transactional Outbox** garantit l'intégrité des messages dans des systèmes distribués.  
En s'appuyant sur les transactions ACID pour sécuriser les opérations locales et un envoi asynchrone des messages, il assure que les données mises à jour sont toujours accompagnées de messages fiables envoyés aux autres services.  
Ce pattern se différencie des autres par sa simplicité et son fonctionnement non bloquant.  
Attention cependant à gérer efficacement la taille de la table *outbox* et faites en sorte que le Message Relay ne deviennent un SPOF pour votre système.

### Eventual Consistency

Le pattern **Eventual Consistency** (ou *Cohérence Éventuelle*) repose sur l'idée que *les différents réplicas de données peuvent temporairement être incohérents, mais finiront par converger vers un état cohérent après un certain temps*.  
Ce modèle est particulièrement utilisé dans des systèmes où la performance et la disponibilité sont capitales, même si cela entraîne des incohérences temporaires.

Lors de l'édition des stocks dans un environnement avec de multiples points de vente (appelés *réplicas* par la suite), la disponibilité et la rapidité d’accès sont primordiales, mais une petite incohérence temporaire dans les quantités disponibles est tolérable.

Dans un système d'e-commerce :

* Le service de commande crée une commande et publie un message via un message broker pour informer le service de gestion des stocks de la mise à jour des quantités disponibles, comme par exemple la décrémentation du stock.
* Le service de gestion des stocks consomme ce message et applique la mise à jour.  
Cependant, cette mise à jour n'est pas immédiatement reflétée dans tous les réplicas, créant ainsi une incohérence temporaire dans les stocks entre les différents points de vente.
* Si des mises à jour concurrentes arrivent sur les stocks (par exemple, deux commandes simultanées pour le même produit), des règles de réconciliation sont appliquées pour résoudre ces conflits, comme la mise à jour du stock en fonction de l’ordre des messages ou de la dernière mise à jour.
* L’état du stock final est alors consigné et les mises à jour réussies sont enregistrées, ce qui permet d'éviter la duplication des événements et de **garantir que le système converge vers un état cohérent**.

![ECommerce Eventual Consistency](/assets/2025/04/01/EventualConsistency.svg)

**Avantages** :

* **Évolutif et résilient** : le système peut facilement s'adapter à une charge de travail croissante et maintenir une haute disponibilité.
* **Faible latence et des performances élevées** : les mises à jour peuvent se produire rapidement, sans attendre la synchronisation complète des réplicas.  

**Inconvénients** :

* **Difficile à concevoir et à dépanner** : les incohérences temporaires peuvent rendre la logique du système complexe à comprendre et à maintenir.
* **Tolérance aux états intermédiaires inconsistants** : le système doit accepter que des données incohérentes puissent exister temporairement pendant que les réplicas convergent vers un état cohérent.

Vous l'aurez compris, bien que ces deux patterns ne garantissent pas l'atomicité globale comme les patterns de transactions distribuées, ils améliorent la résilience, la disponibilité et la tolérance aux erreurs.  
Grâce à leur gestion asynchrone des événements, ils permettent de maintenir la cohérence des données tout en offrant flexibilité et bonnes performances.  
Vous comprenez désormais pourquoi je les apelle "patterns de fiabilité et de cohérence".

## Et la Blockchain, on en parle ?

La blockchain n'est pas un pattern de transaction distribuée à proprement parler.

*C’est un registre de transactions distribuées, c’est-à-dire que l’information n’est ni centralisée, ni décentralisée, mais plutôt distribuée. La gouvernance des transactions se fait à partir de contrats intelligents, d’ententes entre les deux parties qui s’exécutent. Donc, si les conditions contractuelles ne sont pas remplies, la transaction n’est pas complétée.*

Si je devais la comparer avec les transactions distribuées, je dirais que la blockchain est une approche alternative qui garantit la cohérence et l'intégrité des transactions dans un environnement distribué.

La blockchain ne nécessite pas de coordinateur central ni de mécanisme de rollback. À la place, elle utilise des algorithmes de **consensus distribué** ("*Proof of Work*", "*Proof of Stake*", "*Delegated Proof of Stake*", "*Proof of Autority*"...) pour valider et enregistrer les transactions **de manière immuable** (bien qu'il serait mieux de parler d'*immuabilité probabiliste*).

Ce modèle est particulièrement adapté aux environnements où les participants ne se font pas confiance, comme dans les transactions financières.  
En revanche, il introduit une latence plus élevée, ce qui le rend moins efficace pour les systèmes nécessitant une forte réactivité, comme les microservices transactionnels.

Bien que les patterns de transactions distribuées et la blockchain partagent des objectifs similaires, ils répondent à des besoins différents : les premiers privilégient la rapidité et la flexibilité, tandis que la blockchain garantit la transparence et l’immutabilité au sein d'un réseau décentralisé.

## Conclusion

Les patterns de transactions distribuées comme *2PC*, *3PC*, *SAGA* et *TCC* visent à **garantir l'intégrité des données** dans des systèmes répartis.

*2PC* et *3PC* offrent des solutions très strictes en termes d'atomicité, mais au prix de complexité et de risques de blocage (notamment avec 2PC).  
3PC améliore 2PC en réduisant les risques de blocage, mais il reste assez rigide.

*SAGA* et *TCC*, quant à eux, privilégient la **flexibilité** en permettant une gestion asynchrone des transactions.  
SAGA divise une grande transaction en sous-transactions avec des compensations possibles, tandis que TCC réserve d'abord les ressources, puis les confirme ou les annule selon le déroulement de la transaction.

En parallèle, les patterns de fiabilité et de cohérence, comme *Transactional Outbox* et *Eventual Consistency*, se concentrent sur la **gestion des incohérences temporaires** et la **fiabilité** des systèmes distribués, plutôt que sur l'atomicité stricte.  
Ces patterns permettent une gestion asynchrone des événements et garantissent que les systèmes finissent par converger vers un état cohérent, tout en sacrifiant une cohérence immédiate pour une meilleure performance et une meilleure disponibilité.

En résumé, le choix entre patterns de transactions distribuées et les patterns de fiabilité et de cohérence dépend des besoins spécifiques du système :

* gestion stricte de l'atomicité avec 2PC ou 3PC,
* flexibilité avec Saga ou TCC,
* résilience et tolérance aux incohérences temporaires avec les autres patterns.

Le compromis entre atomicité, performance et résilience est essentiel pour faire le bon choix. J'espère que cet article vous aidera à le faire... ce bon choix !

Une dernière chose. Si vous êtes plutôt architecture monolithique modulaire que architecture microservices, ces patterns sont aussi applicables 😁

## 📖 Liens Utiles

* Cohérence des données
  * [Wikipedia](https://fr.wikipedia.org/wiki/Coh%C3%A9rence_(donn%C3%A9es))
* Propriétés ACID
  * [IBM](https://www.ibm.com/docs/en/cics-tx/11.1?topic=processing-acid-properties-transactions)
* Les Patterns
  * [Red Hat Developer Blog](https://developers.redhat.com/)
  * [Microservice Architecture](https://microservices.io/patterns/)
  * [Medium](https://medium.com/)
* La Blockchain
  * [Gestion](https://www.revuegestion.ca/la-technologie-blockchain-une-revolution-pour-la-chaine-d-approvisionnement)
  * [Coin Academy](https://coinacademy.fr/academie/differents-algorithmes-consensus-blockchain/)
