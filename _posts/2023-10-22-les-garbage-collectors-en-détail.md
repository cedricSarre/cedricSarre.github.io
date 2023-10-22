---
layout: post
title: "La Heap Java pour les nuls, mais pas que... Round 2"
---

![Illustration](/assets/images/gc2.jpg) 
<div align="right"><a href="https://fr.freepik.com/vecteurs-libre/camion-poubelle-vecteur-isole-blanc_11296330.htm#query=camion%20poubelle&position=4&from_view=keyword&track=ais#position=4&query=camion%20poubelle">Image de callmetak</a> sur Freepik</div> 

## Table des matières <!-- omit in toc -->

1. [Introduction](#introduction)
2. [Les Garbage Collectors les plus utilisés aujourd'hui](#les-garbage-collectors-les-plus-utilisés-aujourdhui)
   1. [Concurrent Mark and Sweep (CMS)](#concurrent-mark-and-sweep-cms)
   2. [Garbage First Garbage Collector (G1GC)](#garbage-first-garbage-collector-g1gc)
      1. [YoungGC](#younggc)
      2. [OldGC](#oldgc)
3. [Savoir choisir et monitorer son Garbage Collector](#savoir-choisir-et-monitorer-son-garbage-collector)
   1. [Choix](#choix)
   2. [Monitoring](#monitoring)
      1. [A posteriori](#a-posteriori)
      2. [En direct](#en-direct)
      3. [Autre outil](#autre-outil)
4. [Conclusion - Fixer ses exigences pour mieux choisir son Garbage Collector](#conclusion---fixer-ses-exigences-pour-mieux-choisir-son-garbage-collector)

## Introduction

Dans l'article précédent [La Heap Java pour les nuls, mais pas que...](/2023/10/01/la-heap-pour-les-nuls-mais-pas-que.html), nous avons pu découvrir de façon simplifiée le fonctionnement de la gestion de la Heap par le Garbage Collector ainsi que les différentes phases et les différents algorithmes utilisés.

Dans ce nouvel article, je vous propose de voir en détail le fonctionnement des Garbage Collectors les plus utilisés. Je tenterai également de vous faire comprendre comment choisir le plus adapté à vos besoins.
Et enfin, je vous donnerai une liste non exhaustive des meilleurs outils de monitoring.

## Les Garbage Collectors les plus utilisés aujourd'hui

Concernant les GC "*SerialGC*" et "*ParallelGC*", je vous conseille de lire mon précédent article, car ils fonctionnent de manière assez simple et assez similaire, donc nous ne les aborderons pas.

Voyons plus en détails les deux autres, le CMS et le G1GC.

### Concurrent Mark and Sweep (CMS)

Le *CMS* utilise le même algorithme que celui du *ParallelGC* sur la Young Generation. On n’y reviendra pas.  
Sa principale différence se situe donc au niveau de la collecte des objets déréférencés sur la Old Generation.  
En effet, son but est de minimiser les pauses dues à cette collecte en effectuant la plupart des travaux en parallèle des threads de l'application.  
Normalement, il ne copie ni ne compacte les objets référencés. Ce ramassage est effectué sans déplacer les objets vivants.

Vous aurez besoin de comprendre ce qu’est un *GCRoot*, alors en voici une définition simplifiée : *Point de départ du processus de Garbage Collection* Hum... ce n’est pas très clair.  
Il n’existe pas de liste exhaustive officielle (ou je ne l’ai pas encore trouvée) mais disons que ce sont les variables locales des méthodes Java, les Threads actifs, les classes chargées par le Classloader.

Les différentes phases opérées sur la Old Generation lors d’un MajorGC sont :

* **Initial-Mark** ("StopTheWorld") : marquage des objets référencés dans la Old comme "GC roots" mais aussi des objets référencés par un objet "vivant" dans la Young Generation. Ces temps de pause sont très courts par rapport à des MinorGC.  
![CMS-Initial-Mark](/assets/2023/10/02/01-cms-initial-mark.png)

* **Concurrent-Mark** : parcourt le graph d'objets de la Old Generation et marque de manière transitive tous les objets accessibles depuis les "GC roots" identifiés dans la phase précédente, le tout réalisé en parallèle des threads applicatifs (forte baisse du débit ici). Tous les objets "vivants" de la Old Generation ne peuvent être marqués, puisque cette phase n'est pas une phase "StopTheWorld", permettant ainsi à l'application de modifier les références tout au long de cette phase. On peut le constater dans le schéma ci-dessous par la suppression d'une référence anciennement liée à l'objet courant.  
![CMS-Concurrent-Mark](/assets/2023/10/02/02-cms-concurrent-mark.png)

* **Concurrent-Pre-Clean** : la phase précédente s’exécutant en parallèle de l’application, certaines références ont pu être modifiées (exemple de l'ajout d'une référence depuis l'objet courant dans le schéma ci-dessous), et la JVM a marqué les zones de la Heap (zone appelée "Card"), contenant les objets modifiés, comme "sales" (zone appelée "*Dirty Card*").  
![CMS-Concurrent-Pre-Clean](/assets/2023/10/02/03-cms-concurrent-pre-clean.png)
Ensuite, les objets accessibles depuis ces objets sales sont également marqués et les "*Dirty Cards*" sont considérées comme nettoyées.  
![CMS-Concurrent-Pre-Clean](/assets/2023/10/02/04-cms-concurrent-pre-clean.png)

* **Remark** ("StopTheWorld") : recherche des objets qui n’ont pas été marqués dans la phase précédente en raison de potentielles mises à jour réalisées par les threads applicatifs (ajout d’objets, déréférencement...).
Etant donné que tous les objets de la Old Generation encore référencés ont été marqués, c’est après cette phase que le Garbage Collector va nettoyer tous les objets déréférencés (de la Old Generation).

* **Concurrent-Sweep** : supprime les objets déréférencés de la Old, et ajoute l’espace occupé par ces objets à la liste des espaces vides (qui servira à une future allocation). Les objets encore référencés ne sont pas déplacés.  
![CMS-Concurrent-Sweep](/assets/2023/10/02/05-cms-concurrent-sweep.png)

* **Concurrent-Reset** : réinitialise les structures de données de l’algorithme de GC pour être prêt pour la prochaine exécution.

Un des avantages principal de ce Garbage Collector est le fait que, contrairement au SerialGC et au ParallelGC, il n’attend pas que la Old Generation soit pleine avant de déclencher un MajorGC.
Comment fait-il ? Pour faire simple, il est en mesure d’estimer le temps restant avant que la Old Generation ne soit pleine et le temps nécessaire à l’exécution d’un MajorGC. Il est ainsi capable d’anticiper un nettoyage qui sera moins coûteux.

C’est par ce moyen que le CMS réduit considérablement les temps de pause des applications, mais ses principaux problèmes restent la fragmentation de la Old Generation dû au fait qu'il ne comprenne aucune phase de compression, mais également à sa non-prédictibilité des temps de pauses ("StopTheWorld"), plus particulièrement sur de grosses Heap.

### Garbage First Garbage Collector (G1GC)

L'un des principaux objectifs du G1GC est de rendre prévisibles et configurables la durée et la répartition des "StopTheWorld".

Contrairement aux autres, ce n’est pas un Garbage Collector dit "en temps réel". Nous pouvons lui fixer des objectifs de performances spécifiques. Nous pouvons lui demander que les "StopTheWorld" ne dépassent pas *x* millisecondes dans un intervalle de temps donné de *y* millisecondes. Le G1GC fera alors de son mieux pour atteindre cet objectif avec une forte probabilité (mais sans certitude non plus).

Pour y parvenir, le G1GC s'appuie sur un certain nombre de choses.

Tout d'abord, la Heap n'a pas besoin d'être divisée en régions contiguës Young et Old.  
Au lieu de cela, elle est divisée en maximum 2048 régions, de minimum 1MB à maximum 32MB, soit une taille de Heap comprise entre 2GB et 64GB. Il est important de respecter ces configurations sous peine de dégrader le fonctionnement de ce Garbage Collector.  
Chaque région peut être une région Eden, une région Survivor ou une région Old.  
Bien évidemment, les régions Eden et Survivor constituent la Young Generation, et les régions Old constituent la Old Generation.

![G1GC](/assets/2023/10/02/06-g1gc-global-overview.png)

Pourquoi un tel découpage ? Car il évite de collecter toute la Heap en une seule fois, et il aborde le problème de manière incrémentielle : seul un sous-ensemble de régions (appelé "*Collection-Set*") sera considéré à la fois.  
Toutes les régions de la Young sont collectées lors de chaque pause ("StopTheWorld"), tout en permettant que certaines régions de la Old Generation soient également incluses.

Pendant la phase concurrente, le G1GC estime également la quantité d’objets référencés contenue dans chaque région.  
Cette estimation est utilisée pour construire le *Collection-Set* : les régions qui contiennent le plus de déchets sont collectées en premier. D'où le nom : "Garbage First".

#### YoungGC

Sur la Young Generation, une seule phase principale est opérée :

* **Evacuation Pause** : une fois la Young remplie, les threads de l'application sont arrêtés, et les objets référencés à l'intérieur des régions Young sont copiés dans les régions Survivor, ou dans toute région libre qui devient ainsi Survivor. Les objets des régions Survivor ayant atteint le seuil de survie sont déplacés dans des régions de type Old Generation. Vous l'aurez compris, son fonctionnement est très proche de celui du Garbage Collector *CMS*.

Prenons l'exemple ci-dessous d'une Heap ayant des objets dans les régions Young et Old, au moment du déclenchement d'un YoungGC. La case grise représente la région cible qui contiendra les objets vivants.  
![YoungGC](/assets/2023/10/02/07-g1gc-young-gc.png)

Le résultat après le YoungGC (déplacement et compaction) est le suivant.  
![YoungGC-Final-Result](/assets/2023/10/02/08-g1gc-young-gc.png)

#### OldGC

Les différentes phases opérées sur la Old Generation sont :

* **Initial-Mark** ("StopTheWorld") : processus identique au Garbage Collector *CMS*. Cette phase de marquage est assurée lors des YoungGC/MinorGC. Elle ne fait pas réellement partie d'un OldGC/MajorGC mais elle est importante pour la compréhension de la suite.

* **Root-Region-Scan** : scanne les régions Survivor du *Initial-Mark* à la recherche d'objets référencés dans la Old Generation.

* **Concurrent-Mark** : processus assez similaire au *CMS* qui trouve les objets référencés dans l’ensemble de la Heap, qui eux-mêmes permettront d’identifier les régions qu’il sera préférable de nettoyer pendant une *Evacuation Pause*.

* **Remark** ("StopTheWorld") : termine le marquage des objets référencés dans la Heap en utilisant l’algorithme S.A.T.B ("*Snapshot At The Beginning*") qui est plus rapide que celui utilisé par *CMS*. Cette phase nettoie également les régions vides trouvées lors de la phase précédente.

* **Copy/Cleanup** ("StopTheWorld" mais en parallèle) :
  * Il sélectionne les régions de la Old ayant le moins d’objets référencés et les régions ayant la plus courte durée de vie, car ces deux types de régions sont les plus rapides à nettoyer.
  * Ces régions sont ensuite nettoyées lors d’un YoungGC : on appelle ça le processus d’*Evacuation Mixte*, permettant ainsi de nettoyer en même temps la Young et la Old. Les régions collectées, mais non nettoyées, sont ensuite compactées.

Pour conclure, le G1GC pallie certains problèmes de l’algorithme *CMS*, par la prévisibilité des "StopTheWorld" et en terminant son processus par la fragmentation de la Heap.

Si votre application n’est pas limitée par l’utilisation CPU, mais qu’elle est très sensible à la latence, alors le CMS ou le G1GC est sans doute le meilleur choix.

Si au contraire, votre application est liée au débit ou consomme 100% du CPU et que les "StopTheWorld" ne sont pas vraiment un problème, alors le ParallelGC peuvent s’avérer être un meilleur choix.

Dans tous les cas, afin de réaliser le meilleur choix et de configurer au mieux votre GC, il est recommandé de tester et de monitorer votre application.

Bien évidemment, il existe d'autres Garbage Collectors, mais nous ne les aborderons pas en détail dans cet article. Dans le prochain ?

* **ZGC** : effectue simultanément tous les travaux coûteux, sans arrêter l’exécution des threads de l’application pendant plus de 10 millisecondes, ce qui le rend approprié à des applications nécessitant une faible latence et/ou utilisant une très grande Heap.

* **Epsilon GC** : alloue la mémoire, mais ne collecte pas les objets déréférencés. Il est très approprié pour traitements par lot d'assez courte durée.

* **Shenandoah GC** : réduit les temps de pause GC en effectuant plus de travail de collecte d'objets déréférencés en même temps que le programme Java en cours d’exécution. Le CMS et le G1 effectuent tous deux le marquage simultané des objets vivants. Shenandoah ajoute le compactage simultané. Il utilise des zones de mémoire pour gérer les objets qui ne sont plus utilisés, et ceux qui sont vivants et prêts à être compactés.

## Savoir choisir et monitorer son Garbage Collector

Pour choisir le Garbage collector le plus adapté à son application, pas de mystère, il faut déjà savoir le configurer et le monitorer.  
Pourquoi le monitorer ? Au fil des évolutions de votre application, il se peut que vous ayez à modifier le paramétrage ou à changer de Garbage Collector. Il est donc important de pouvoir l'anticiper.  
Votre besoin est aussi un élément qui doit guider votre choix. Ai-je besoin d'un fort débit, d'une faible latence, ou des deux ?

### Choix

Commençons par voir la liste non-exhaustive des paramètres communs à tous les Garbage Collectors les plus utilisés :

* **-Xmx** : expace mémoire maximal qui pourra être allouée à la Heap tout au long se vie.
* **-Xms** : espace mémoire alloué à la Heap au démarrage de la JVM.
* **-XX:NewRatio** : rapport entre la taille de la Young Generation et de la Old Generation. Par défaut, ce ratio est égal à 2 : 1/3 de l'espace mémoire est affecté à la Young Generation et 2/3 sont affectés à la Old Generation.
* **-XX:SurvivorRatioSetting** : rapport entre la taille de l'Eden et des Survivors. Par défaut, ce ratio est égale à 8 : 6/8 de l'espace mémoire est affecté à l'Eden et 1/8 est affecté à chacun des 2 Survivors.

Oracle [recommande](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/sizing.html) de configurer les valeurs des paramètres *Xmx* et *Xms* avec les mêmes valeurs, permettant ainsi d'allouer dès son démarrage toute la mémoire estimée nécessaire à la JVM. Ceci permettrait d'éviter les allocations et des désallocations mémoire coûteuses et donc, en principe d'améliorer les performances.

Le Garbage Collector par défaut depuis Java 9 est le G1GC.  
Pour rappel, ce dernier a pour but de respecter le temps de pause configuré et il y arrive en ajustant la taille de la Young Generation (Eden et Survivors). Je vous recommande donc de configurer le temps de pause maximal que G1GC essaiera de respecter via le paramètre **-XX:MaxGCPauseMillis**. Par défaut, il est égal à 200 millisecondes.  
Vous pouvez également configurer le nombre d'itérations (ou l'âge) pour conserver les objets vivants dans la Young Generation : rappelez-vous, ce nombre d'itérations équivaut au nombre de déplacements d'un objet entre les différents Survivors avant d'être déplacé dans la Old Generation. Pour cela, utilisez le paramètre **-XX:MaxTenuringThreshold** (par défaut à 15).

De nombreux autres paramètres existent, mais dans un premier temps, ceux-ci devraient vous suffire.

Comme nous l'avons vu, le choix du Garbage Collector dépend de vos besoins en termes de débit, de latence, mais également en fonction de vos ressources.

Pour utiliser le SerialGC, utilisez l'option **-XX:+UseSerialGC**.  
Pour utiliser le ParallelGC (utilisé par défaut en Java8), utilisez l'option **-XX:+UseParallelGC**.  
Pour utiliser le CMS (déprécié depuis Java 9), utilisez l'option **-XX:+UseConcMarkSweepGC**.  
Enfin, pour utiliser le G1GC (utilisé par défaut depuis Java 9), utilisez l'option **-XX:+UseG1GC**.  
Quelque soit votre version de Java, il y a un Garbage Collector par défaut. Je vous conseille de configurer tout de même celui de votre choix, vous éviterez ainsi certaines surprises lors de vos montées de version.

### Monitoring

#### A posteriori

Quoi de mieux pour monitorer que des logs ? Et Java nous fournit ce que l'on appelle les logs GC.  
*Les logs c'est bien*, me direz-vous, *mais leur activation ne va-t-elle pas dégrader les performances de mon application ?* Plusieurs benchmarks ont été réalisés et la réponse est toujours la même : *Aucune différence notable en termes de consommation CPU et mémoire*. Donc ne vous privez pas de ces derniers !

Different outils existent pour analyser ces logs. Mais ma préférence personnelle est l'utilisation de [GCeasy](https://gceasy.io/).
De plus, le site Web de GCEasy est très bien fait et vous pourrez y trouver un grand nombre d'[articles](https://blog.gceasy.io/).

#### En direct

Différents outils existent pour suivre au fil de l'eau le comportement de la JVM :

* [JVisualVM](https://visualvm.github.io/)
* [JProfiler](https://www.ej-technologies.com/products/jprofiler/overview.html)
* [YourKit](https://www.yourkit.com/java/profiler/features/)
* [JavaMissionControl](https://www.oracle.com/java/technologies/jdk-mission-control.html)

Ceux-ci sont très utiles lors des développements et/ou lors des benchmarks, afin d'identifier de potentielles fuites mémoire, et de configurer au mieux sa Heap.  
Etant donné qu'ils requièrent une connexion directe à la JVM, ils sont donc peu utilisés sur une plateforme de production.

#### Autre outil

[Eclipe MAT](https://www.eclipse.org/mat/) (Memory Analysis Tool) vous permettra d'analyser un Heap Dump, ce dernier étant une photo de la Heap à un instant t (suite à un FullGC). Il peut être déclenché manuellement ou de manière automatique lors d'un OutOfMemory (option **-XX:+HeapDumpOnOutOfMemoryError**).

## Conclusion - Fixer ses exigences pour mieux choisir son Garbage Collector

Pour écrire cet article, je me suis beaucoup appuyé sur mon expérience et mes nombreuses lectures de GCLogs, mais aussi et principalement sur la documentation d'Oracle pour les Garbage Collectors abordés : le [G1GC](https://www.oracle.com/technetwork/tutorials/tutorials-1876574.html) et le [CMS](https://docs.oracle.com/javase/9/gctuning/toc.htm).

Vous l'aurez compris, le choix du Garbage Collector le plus adapté à votre application est difficile. Etant donné les performances du G1GC, on se pose de moins en moins la question de choisir le meilleur GC. Que vous gardiez celui par défaut ou non, je vous conseille tout de même d'activer les logs GC, de configurer la JVM pour générer un HeapDump en cas de crash de celle-ci et surtout de suivre l'évolution de votre Heap.

Je le répète souvent aux équipes : on peut garder le Garbage Collector par défaut durant le cycle de développements mais, avant de mettre votre produit en production, choisir et configurer le Garbage Collector le plus adapté à vos exigences est essentiel : prioriser le débit pour un batch, et prioriser une latence faible pour une application Web. Sans oublier évidemment l'empreinte mémoire, **capitale** à l'heure du GreenIT.
