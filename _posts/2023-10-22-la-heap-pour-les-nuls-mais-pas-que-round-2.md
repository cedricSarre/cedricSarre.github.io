---
layout: post
title: "La Heap Java pour les nuls, mais pas que... Round 2"
resume: "Dans ce nouvel article, je vous propose de voir en détail le fonctionnement des Garbage Collectors les plus utilisés, CMS et G1GC."
image: "/assets/images/gc2.jpg"
imageCopyright: "https://fr.freepik.com/vecteurs-libre/camion-poubelle-vecteur-isole-blanc_11296330.htm#query=camion%20poubelle&position=4&from_view=keyword&track=ais#position=4&query=camion%20poubelle"
imageDe: Image de callmetak sur Freepik
readTime: "45"
excerpt: Heap Java Garbage Collector G1GC
---

## Table des matières <!-- omit in toc -->

1. [Introduction](#introduction)
2. [Les deux Garbage Collectors les plus utilisés aujourd'hui](#les-deux-garbage-collectors-les-plus-utilisés-aujourdhui)
   1. [Concurrent Mark and Sweep (CMS)](#concurrent-mark-and-sweep-cms)
   2. [Garbage First Garbage Collector (G1GC)](#garbage-first-garbage-collector-g1gc)
      1. [MinorGC](#minorgc)
      2. [MajorGC](#majorgc)
3. [Savoir choisir et monitorer son Garbage Collector](#savoir-choisir-et-monitorer-son-garbage-collector)
   1. [Choix](#choix)
   2. [Monitoring](#monitoring)
      1. [A posteriori](#a-posteriori)
         1. [Les GCLogs](#les-gclogs)
         2. [Eclipse MAT](#eclipse-mat)
      2. [En direct](#en-direct)
4. [Exemple de Netflix](#exemple-de-netflix)
5. [Fixer ses exigences pour mieux choisir son Garbage Collector](#fixer-ses-exigences-pour-mieux-choisir-son-garbage-collector)

## Introduction

Dans l'article précédent [La Heap Java pour les nuls, mais pas que...](https://cedricsarre.github.io/2023/10/01/la-heap-pour-les-nuls-mais-pas-que.html), nous avons pu découvrir de façon simplifiée le fonctionnement de la gestion de la Heap par le Garbage Collector ainsi que les différentes phases et les différents algorithmes utilisés.

Dans ce nouvel article, nous vous proposons de voir en détail le fonctionnement de deux des Garbage Collectors les plus utilisés. Nous tenterons également de vous faire comprendre comment choisir le plus adapté à vos besoins.
Et enfin, nous vous donnerons une liste non exhaustive des meilleurs outils de monitoring.

## Les deux Garbage Collectors les plus utilisés aujourd'hui

Concernant "*SerialGC*" et "*ParallelGC*" (le Garbage Collector par défaut de Java 8), nous vous conseillons de lire notre précédent article, car ils fonctionnent de manière assez simple et assez similaire, donc nous ne les aborderons pas.

Voyons plus en détails les deux autres, le CMS et le G1GC (le Garbage Collector par défaut depuis Java 9).

### Concurrent Mark and Sweep (CMS)

Tout comme SerialGC et ParallelGC, CMS utilise l'algorithme Mark-Copy sur la Young Generation, mais en parallèle. Contrairement à "Mark-Copy" qui utilise un seul thread pour les opérations de marquage et de copie, "Parallel-Mark-Copy" utilise plusieurs threads pour exécuter ces opérations en parallèle. Cela permet d'accélérer le processus de Garbage Collection en utilisant efficacement les ressources CPU disponibles.
La principale différence de CMS se situe donc au niveau de la collecte des objets déréférencés sur la Old Generation.  
En effet, son but est de minimiser les pauses dues à cette collecte en effectuant la plupart des travaux en parallèle des threads de l'application.  
Il ne copie ni ne compacte les objets référencés. Ce ramassage est effectué sans déplacer les objets vivants.

Le GC root est le point de départ du processus de collecte. C'est un arbre de dépendance de tous les objets java (classe, instances, threads, ...) actuellement utilisés par la JVM.
Il s'agit donc des objets qui ne doivent pas être collectés par le processus de GC - variables locales des méthodes Java, Threads actifs et classes chargées par le Classloader - puisque toujours référencés par l'application.

Les différentes phases opérées sur la Old Generation lors d’un "MajorGC" sont :

* **Initial-Mark** (Stop-The-World) : marquage des objets référencés dans la Old comme "GC roots" mais aussi des objets référencés par un objet "vivant" dans la Young Generation. Ces temps de pause sont très courts par rapport à des "MinorGC".  
![CMS-Initial-Mark](/assets/2023/10/22/01-cms-initial-mark.png)

* **Concurrent-Mark** : parcourt le graphe d'objets de la Old Generation et marque de manière transitive tous les objets accessibles depuis les "GC roots" identifiés dans la phase précédente, le tout réalisé en parallèle des threads applicatifs (forte baisse du débit ici). Tous les objets "vivants" de la Old Generation ne peuvent être marqués, puisque cette phase n'est pas une phase Stop-The-World, permettant ainsi à l'application de modifier les références tout au long de cette phase. On peut le constater dans le schéma ci-dessous par la suppression d'une référence anciennement liée à l'objet courant.  
![CMS-Concurrent-Mark](/assets/2023/10/22/02-cms-concurrent-mark.png)

* **Concurrent-Pre-Clean** : la phase précédente s’exécutant en parallèle de l’application, certaines références ont pu être modifiées (exemple de l'ajout d'une référence depuis l'objet courant dans le schéma ci-dessous), et la JVM a marqué les zones de la Heap (zone appelée "Card"), contenant les objets modifiés, comme "sales" (zone appelée "*Dirty Card*").  
![CMS-Concurrent-Pre-Clean](/assets/2023/10/22/03-cms-concurrent-pre-clean.png)  
Ensuite, les objets accessibles depuis ces objets sales sont également marqués et les "*Dirty Cards*" sont considérées comme nettoyées.  
![CMS-Concurrent-Pre-Clean](/assets/2023/10/22/04-cms-concurrent-pre-clean.png)

* **Remark** (Stop-The-World) : recherche des objets qui n’ont pas été marqués dans la phase précédente en raison de potentielles mises à jour réalisées par les threads applicatifs (ajout d’objets, déréférencement...).
Etant donné que tous les objets de la Old Generation encore référencés ont été marqués, c’est après cette phase que le Garbage Collector va nettoyer tous les objets déréférencés (de la Old Generation).

* **Concurrent-Sweep** : supprime les objets déréférencés de la Old, et ajoute l’espace occupé par ces objets à la liste des espaces vides, qui servira à une future allocation. Les objets encore référencés ne sont pas déplacés.  
![CMS-Concurrent-Sweep](/assets/2023/10/22/05-cms-concurrent-sweep.png)

* **Concurrent-Reset** : réinitialise les structures de données de l’algorithme de GC pour être prêt pour la prochaine exécution.

Un des avantages principaux de ce Garbage Collector est le fait que, contrairement au SerialGC et au ParallelGC, il n’attend pas que la Old Generation soit pleine avant de déclencher un "MajorGC".
Comment fait-il ? Pour faire simple, il est en mesure d’estimer le temps restant avant que la Old Generation ne soit pleine et le temps nécessaire à l’exécution d’un "MajorGC". Il est ainsi capable d’anticiper un nettoyage qui sera moins coûteux.

C’est par ce moyen que le CMS réduit considérablement les temps de pause des applications, mais ses principaux problèmes restent la fragmentation de la Old Generation dû au fait qu'il ne comprenne aucune phase de compression, mais également à sa non-prédictibilité des temps de pauses (Stop-The-World), plus particulièrement sur de grosses Heap.

### Garbage First Garbage Collector (G1GC)

L'un des principaux objectifs du G1GC est de rendre prévisibles et configurables la durée et la répartition des Stop-The-World.

Contrairement aux autres, ce n’est pas un Garbage Collector dit "en temps réel". Nous pouvons lui fixer des objectifs de performances spécifiques. Nous pouvons lui demander que les Stop-The-World ne dépassent pas *x* millisecondes dans un intervalle de temps donné de *y* millisecondes. Le G1GC fera alors de son mieux pour atteindre cet objectif avec une forte probabilité (mais sans certitude non plus).

Pour y parvenir, le G1GC s'appuie sur un certain nombre de principes.

Tout d'abord, la Heap n'a pas besoin d'être divisée en régions contiguës Young et Old.  
Au lieu de cela, elle est divisée en maximum 2048 régions, de minimum 1MB à maximum 32MB, soit une taille de Heap comprise entre 2GB et 64GB. Il est important de respecter ces configurations sous peine de dégrader le fonctionnement de ce Garbage Collector.  
Chaque région peut être une région Eden, une région Survivor ou une région Old.  
Bien évidemment, les régions Eden et Survivor constituent la Young Generation, et les régions Old constituent la Old Generation.

![G1GC](/assets/2023/10/22/06-g1gc-global-overview.png)

Pourquoi un tel découpage ? Car il évite de collecter toute la Heap en une seule fois, et il aborde le problème de manière incrémentielle : seul un sous-ensemble de régions (appelé *Collection-Set*) sera considéré à la fois.  
Toutes les régions de la Young sont collectées lors de chaque pause (Stop-The-World), tout en permettant que certaines régions de la Old Generation soient également incluses.

Pendant la phase concurrente, le G1GC estime également la quantité d’objets référencés contenue dans chaque région.  
Cette estimation est utilisée pour construire le *Collection-Set* : les régions qui contiennent le plus de déchets sont collectées en premier. D'où le nom : "Garbage First".

#### MinorGC

Sur la Young Generation, une seule phase principale est opérée :

* **Evacuation Pause** : une fois la Young Generation remplie, les threads de l'application sont arrêtés, et les objets référencés à l'intérieur des régions Young sont copiés dans les régions Survivor, ou dans toute région libre qui devient ainsi Survivor.  
Les objets des régions Survivor ayant atteint le seuil de survie sont déplacés dans des régions de type Old Generation pour libérer de l'espace dans les régions Survivor.  
Cette phase, similaire au fonctionnement du Garbage Collector CMS, consiste à déplacer les objets vivants des régions Young vers les régions Survivor ou directement vers la Old Generation, contribuant ainsi à minimiser les temps d'arrêt de l'application.

Prenons l'exemple ci-dessous d'une Heap ayant des objets dans les régions Young et Old, au moment du déclenchement d'un "MinorGC". La case grise représente la région cible qui contiendra les objets vivants.  
![MinorGC](/assets/2023/10/22/07-g1gc-young-gc.png)

Le résultat après le "MinorGC" (déplacement et compaction) est le suivant.  
![MinorGC-Final-Result](/assets/2023/10/22/08-g1gc-young-gc.png)

#### MajorGC

Les différentes phases opérées sur la Old Generation sont :

* **Initial-Mark** (Stop-The-World), processus identique au Garbage Collector *CMS*, et bien que techniquement réalisé lors des Minor GC, il est fondamental pour établir une base de référence lors de chaque Major GC.  
![Initial-Mark](/assets/2023/10/22/09-g1gc-old-gc-initial-mark.png)

* **Root-Region-Scan** scanne les régions Survivor du *Initial-Mark* à la recherche d'objets référencés dans la Old Generation.

* **Concurrent-Mark**, processus assez similaire au *CMS*, qui trouve les objets référencés dans l’ensemble de la Heap, qui eux-mêmes permettront d’identifier les régions qu’il sera préférable de nettoyer pendant une *Evacuation Pause*. Cette phase garantit ainsi que la collecte peut être réalisée de manière efficace et prévisible.  
![Concurrent-Mark](/assets/2023/10/22/10-g1gc-old-gc-concurrent-mark.png)

* **Remark** (Stop-The-World) termine le marquage des objets référencés dans la Heap en utilisant l’algorithme S.A.T.B ("*Snapshot At The Beginning*") qui est plus rapide que celui utilisé par *CMS*, garantissant ainsi que tous les objets vivants ont été correctement identifiés. De plus, cette phase nettoie également les régions vides trouvées lors des phases précédentes.  
![Remark](/assets/2023/10/22/11-g1gc-old-gc-remark.png)

* **Copy/Cleanup** (Stop-The-World mais de façon concurrente) sélectionne les régions de la Old Generation ayant le moins d'objets référencés et les régions ayant la plus courte durée de vie. Ces régions sont nettoyées en parallèle lors d'un MinorGC, dans un processus appelé *Evacuation Mixte*. Cela permet de nettoyer simultanément la Young et la Old Generation. Les régions collectées mais non nettoyées sont ensuite compactées.  
![Avant Copy/Cleanup](/assets/2023/10/22/12-g1gc-old-gc-copy-cleanup.png)  
La compaction consiste à déplacer les objets vivants vers des régions contiguës, réduisant ainsi la fragmentation de la mémoire et améliorant l'utilisation de l'espace. Cela permet d'optimiser les performances de la JVM en minimisant les temps d'accès à la mémoire et en évitant la fragmentation excessive.  
![Après Copy/Cleanup](/assets/2023/10/22/13-g1gc-old-gc-after-copy-cleanup.png)

Pour conclure, le *G1GC* résout certains problèmes liés à l'algorithme *CMS* en offrant une meilleure prévisibilité des temps d'arrêt (Stop-The-World) et en gérant plus efficacement la fragmentation de la Heap.

Le choix entre le *CMS*, le *G1GC* et le *ParallelGC* dépend des caractéristiques spécifiques de votre application. Si votre application est très sensible à la latence et que la prévisibilité des temps d'arrêt est essentielle, alors le CMS ou le G1GC est probablement le meilleur choix. En revanche, si votre application est liée au débit ou utilise pleinement le CPU sans être gênée par les temps d'arrêt, le ParallelGC peut être plus adapté.

Bien évidemment, il existe d'autres Garbage Collectors, mais nous ne les aborderons pas en détail dans cet article.

* **ZGC** : effectue simultanément tous les travaux coûteux, sans arrêter l’exécution des threads de l’application pendant plus de 10 millisecondes, ce qui le rend approprié à des applications nécessitant une faible latence et/ou utilisant une très grande Heap.

* **Epsilon GC** : alloue la mémoire, mais ne collecte pas les objets déréférencés. Il est très approprié pour les traitements par lots d'assez courte durée.

* **Shenandoah GC** : réduit les temps de pause GC en effectuant plus de travail de collecte d'objets déréférencés en même temps que le programme Java en cours d’exécution. Le CMS et le G1 effectuent tous deux le marquage simultané des objets vivants. Shenandoah ajoute le compactage simultané. Il utilise des zones de mémoire pour gérer les objets qui ne sont plus utilisés, et ceux qui sont vivants et prêts à être compactés.

## Savoir choisir et monitorer son Garbage Collector

Pour choisir le Garbage collector le plus adapté à son application, pas de mystère, il faut déjà savoir le configurer et le monitorer.  
Pourquoi le monitorer ? Au fil des évolutions de votre application, il se peut que vous ayez à modifier le paramétrage ou à changer de Garbage Collector. Il est donc important de pouvoir l'anticiper.  
Votre besoin est aussi un élément qui doit guider votre choix. Ai-je besoin d'un fort débit, d'une faible latence, ou des deux ?

### Choix

Commençons par voir les options de la JVM à configurer absolument :

* "**-Xmx=**" : espace mémoire maximal qui pourra être allouée à la Heap tout au long sa vie.
* "**-Xms=**" : espace mémoire alloué à la Heap au démarrage de la JVM.
* "**-XX:MetaspaceSize=**" : espace mémoire allouée au Metaspace (ne fait pas partie de l'espace mémoire géré par l'option *-Xmx*) au démarrage de la JVM.
* "**-XX:MaxMetaspaceSize=**" : espace mémoire maximal qui pourra être alloué au MetaSpace.

Oracle [recommande](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/sizing.html) de configurer les valeurs des paramètres *Xmx* et *Xms* avec les mêmes valeurs, permettant ainsi d'allouer dès son démarrage toute la mémoire estimée nécessaire à la JVM. Ceci permettrait d'éviter des allocations et des désallocations mémoire coûteuses et donc, en principe d'améliorer les performances.

Le Metaspace est un espace mémoire utilisé par la JVM pour stocker les métadonnées des classes chargées dynamiquement ainsi que les informations relatives aux méthodes, aux constantes et aux autres éléments de structure de ces classes.  
Contrairement à la mémoire de la Heap qui est destinée au stockage des objets instantiés, le Metaspace est dédié au stockage des données concernant la structure et les comportements des classes. À partir de Java 8, le Metaspace a remplacé la zone de mémoire permanente (PermGen) qui était utilisée dans les versions antérieures de Java pour stocker ces métadonnées. L'avantage du Metaspace est sa gestion dynamique de la mémoire, ce qui évite les problèmes liés à la taille limitée de la zone de mémoire permanente.

Sur les anciens Garbage Collectors (Serial, Parallel et Concurrent), ces deux autres options peuvent s'avérer utiles :

* "**-XX:NewRatio=**" : rapport entre la taille de la Young Generation et de la Old Generation. Par défaut, ce ratio est égal à 2 : 1/3 de l'espace mémoire est affecté à la Young Generation et 2/3 sont affectés à la Old Generation.
* "**-XX:SurvivorRatioSetting=**" : rapport entre la taille de l'Eden et des Survivors. Par défaut, ce ratio est égale à 8 : 6/8 de l'espace mémoire est affecté à l'Eden et 1/8 est affecté à chacun des 2 Survivors.

G1GC a pour but de respecter le temps de pause configuré et il y arrive en ajustant la taille de la Young Generation (Eden et Survivors). Nous vous recommandons donc de configurer le temps de pause maximal que G1GC essaiera de respecter via le paramètre "**-XX:MaxGCPauseMillis=**". Par défaut, il est égal à 200 millisecondes. Fixer une valeur trop basse risque d'entraîner des pauses plus fréquentes du GC, ce qui peut ralentir l'exécution de l'application. D'un autre côté, une valeur trop élevée prolongera les temps de pause du GC, potentiellement au détriment de l'expérience utilisateur. Il est donc essentiel de trouver un équilibre entre des temps de pause courts pour maintenir la réactivité de l'application et des temps suffisamment longs pour permettre au GC de terminer efficacement son travail.

Vous pouvez également configurer le nombre d'itérations (ou l'âge) pour conserver les objets vivants dans la Young Generation : rappelez-vous, ce nombre d'itérations équivaut au nombre de déplacements d'un objet entre les différents Survivors avant d'être déplacé dans la Old Generation. Pour cela, utilisez le paramètre "**-XX:MaxTenuringThreshold=**" (par défaut à 15). Fixer une valeur trop basse pour ce paramètre peut entraîner une promotion prématurée des objets vers la Old Generation, augmentant ainsi la charge de travail du GC et potentiellement entraîner une fragmentation de la Heap. À l'inverse, une valeur trop élevée peut retarder la promotion des objets vivants vers la Old Generation, ce qui peut augmenter la taille de la Young Generation et potentiellement affecter les performances de l'application en raison d'une utilisation excessive de la mémoire. Trouver un équilibre entre ces deux extrêmes est donc essentiel pour optimiser l'utilisation de la mémoire et les performances de l'application avec le G1GC.

Nous vous conseillons également de configurer les options suivantes :

* "**-XX:+UseStringDeduplication**" : une étude récente a montré que 13.5% de la mémoire d'une application est de la duplication de chaînes de caractères (String). Donc 13.5% de notre mémoire applicative est gaspillée.  
En utilisant cette option, la JVM essaye alors de supprimer les duplications de *String* durant un cycle de *Garbage Collection*. Ce travail pouvant avoir un coût en termes de ressources CPU, il est donc appliqué aux objets *String* ayant une durée de vie assez longue (ayant survécu à plusieurs "MinorGC").
* "**-XX:+StringDeduplicationAgeThreshold=**" : permet de préciser à partir combien de cycles du Garbage Collector un *String* est éligible à une déduplication.

Pour utiliser le SerialGC, utilisez l'option "**-XX:+UseSerialGC**".  
Pour utiliser le ParallelGC, utilisez l'option "**-XX:+UseParallelGC**".  
Pour utiliser le CMS (déprécié depuis Java 9), utilisez l'option "**-XX:+UseConcMarkSweepGC**".  
Enfin, pour utiliser le G1GC (utilisé par défaut depuis Java 9), utilisez l'option "**-XX:+UseG1GC**".  
Nous vous conseillons de configurer tout de même celui de votre choix, même s'il s'agit du Garbage Collector par défaut, vous éviterez ainsi certaines surprises lors de vos montées de version de Java.

De nombreux autres paramètres existent, mais dans un premier temps, ceux-ci devraient vous suffire.

### Monitoring

Comme nous l'avons vu, le choix du Garbage Collector dépend de vos besoins en termes de débit, de latence, mais également en fonction de vos ressources, d'où le besoin fort de monitoring.

#### A posteriori

##### Les GCLogs

Quoi de mieux pour monitorer que des logs ? Et Java nous fournit ce que l'on appelle les *GCLogs* (logs du Garbage Collector).  
"*Les logs c'est bien*", me direz-vous, "*mais leur activation ne va-t-elle pas dégrader les performances de mon application ?*"  
Plusieurs benchmarks ont été réalisés et la réponse est toujours la même : "*Aucune différence notable en termes de consommation CPU et mémoire*". Donc ne vous privez pas de ces derniers !

Il est évident que l'impact sur les performances va dépendre du niveau de log que vous choisirez. En utilisant le niveau **INFO**, peu de chance en effet de dégrader les performances. Si vous souhaitez utiliser le niveau **DEBUG** ou **TRACE**, privilégiez-les sur des environnements de benchmarks ou de pré-production.  
Vous pouvez également activer ce genre de niveau de logs très détaillé en production, mais plutôt sur une courte durée à des fins d'analyses.

Differents outils existent pour analyser ces logs. Ma préférence personnelle est l'utilisation de [GCeasy](https://gceasy.io/). Le site Web est très bien fait et vous pourrez y trouver un grand nombre d'[articles](https://blog.gceasy.io/) et de conseils.

Comment activer les gclogs ? Simplement via l'option de la JVM suivante : `-Xlog:gc*=info:file=/tmp/gclogs/gc.log:time,uptime,level,tags:filecount=10,filesize=20m`.

* *-Xlog* : génération des logs du Garbage collector.
* *gc\*=info* : toutes les phases du GC de niveau *info* sont tracées. Pour logguer moins, il suffit de retirer l'étoile. Pour logguer plus, il suffit de changer le niveau de log.
* *file=* : chemin et nom du fichier des logs gc.
* *filecount* : nombre maximum de fichiers de logs gc (tournants).
* *filesize* : taille maximale des fichiers de logs gc (ici en MB).

Cette commande générera donc au maximum 10 fichiers de 20MB, soit 200MB de logs. Lorsque vous activer cette option, choisissez bien ces 2 valeurs et soyez sûrs d'avoir l'espace disque suffisant.

##### Eclipse MAT

[Eclipe MAT](https://www.eclipse.org/mat/) (Memory Analysis Tool) vous permettra d'analyser un *HeapDump*, ce dernier étant une photo de la Heap à un instant *t*.  

La génération d'un *HeapDump* peut être déclenchée manuellement, auquel cas, un "FullGC" est déclenché avant le dump, mais attention, votre application sera freezée durant toute la durée du dump. Cette opération est peu recommandée sur une plateforme de production mais peut s'avérer très utile lors d'un benchmark.

Cette génération peut également être déclenchée de manière automatique lors de détection d'une erreur de type *OutOfMemoryError* (option **-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp/gclogs/**).

Attention cependant à l'espace disque disponible. En effet, le *HeapDump* étant une image de la Heap, lorsqu'il est déclenché automatiquement sur l'apparition d'un *OutOfMemory*, le fichier de dump aura alors la même taille que la heap. Si celle-ci fait 8GB, le HeapDump généré fera lui aussi 8GB.

#### En direct

Différents outils existent pour suivre au fil de l'eau le comportement de la JVM :

* [JVisualVM](https://visualvm.github.io/) :  outil de surveillance, de profilage et d'analyse des performances. Il est inclus dans la distribution standard de Java Development Kit (JDK).
* [JProfiler](https://www.ej-technologies.com/products/jprofiler/overview.html) : outil de profilage avancé (mémoire, CPU, threads, requêtes SQL, appels de méthodes...) qui permet d'analyser les performances et de diagnostiquer les problèmes de manière détaillée. Il est disponible en tant qu'outil autonome ou en tant que plugin dans certains IDE tels que IntelliJ.
* [YourKit](https://www.yourkit.com/java/profiler/features/) : outil de profilage (mémoire, CPU, threads et requêtes SQL) hautement performant qui permet d'analyser les performances et de diagnostiquer les problèmes. Il est disponible en tant qu'outil autonome ou en tant que plugin dans certains IDE tels que IntelliJ.
* [JavaMissionControl](https://www.oracle.com/java/technologies/jdk-mission-control.html) : outil avancé de surveillance, de profilage et de gestion des performances. Il offre une interface graphique pour surveiller et analyser les performances de l'application en temps réel, ainsi que pour diagnostiquer les problèmes de performance. Il est fourni en tant que composant de Java Flight Recorder (JFR) dans le JDK et peut être utilisé en conjonction avec d'autres outils de diagnostic

Ceux-ci sont très utiles lors des développements et/ou lors des benchmarks, afin d'identifier de potentielles fuites mémoire, et de configurer au mieux sa Heap.  
Etant donné qu'ils requièrent une connexion directe à la JVM, ils sont moins utilisés sur une plateforme de production, souvent à cause de potentiels problèmes de sécurité (liés à l'ouverture nécessaire de ports).

## Exemple de Netflix

Vous l'aurez compris, le choix du Garbage Collector le plus adapté à votre application est difficile. Etant donné les performances du G1GC, on se pose de moins en moins la question de choisir le meilleur GC.  

Mais le parfait contre-exemple a été publié par Netflix dans un [article dédié](https://netflixtechblog.com/bending-pause-times-to-your-will-with-generational-zgc-256629c9386b). Et il semblerait que migrer du G1GC à ZGC a considérablement augmenté leurs performances applicatives, comme mentionné en toute fin de leur article.

> Il n'y a pas de Garbage Collector meilleur qu'un autre, il faut choisir le plus adapté.  
Le passage à ZGC par défaut a fourni aux responsables applicatifs l'occasion idéale de réfléchir à leur choix de Garbage Collector. Plusieurs cas de traitement par lots ou de précalculs utilisaient G1GC par défaut, alors qu'ils auraient bénéficié d'un meilleur débit grâce au ParallelGC.  
Durant une grande charge de travail de précalcul, nous avons constaté une amélioration de 6 à 8 % du débit de l'application, ce qui a permis d'économiser une heure sur le temps de traitement par lots, par rapport à G1GC.

## Fixer ses exigences pour mieux choisir son Garbage Collector

Que vous gardiez celui par défaut ou non, je vous conseille tout de même d'activer les logs du Garbage Collector, de configurer la JVM pour générer un *HeapDump* en cas de *OutOfMemoryError* et surtout de suivre l'évolution de votre Heap. Donc monitorez !

Je le répète souvent aux équipes : **on peut garder le Garbage Collector par défaut durant le cycle de développements mais, avant de mettre le produit en production, il est essentiel que vous déterminiez et que vous configuriez le Garbage Collector le plus adapté aux exigences** : prioriser le débit pour un batch, et prioriser une latence faible pour une application Web. Sans oublier évidemment l'empreinte mémoire et la consommation CPU, **capitales** à l'heure du GreenIT.

*Documentations officielles :*

* [Documentation Oracle du G1GC](https://www.oracle.com/technetwork/tutorials/tutorials-1876574.html)
* [Documentation Oracle du CMS](https://docs.oracle.com/javase/9/gctuning/toc.htm)