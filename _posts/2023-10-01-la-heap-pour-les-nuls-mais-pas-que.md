---
layout: post
title: "La Heap Java pour les nuls, mais pas que..."
---

## Table des matières <!-- omit in toc -->

1. [Introduction](#introduction)
2. [Le fonctionnement très simplifié de la Heap](#le-fonctionnement-très-simplifié-de-la-heap)
3. [Le fonctionnement plus détaillé de la Heap](#le-fonctionnement-plus-détaillé-de-la-heap)
4. [Les algorithmes au sein des Garbage Collectors](#les-algorithmes-au-sein-des-garbage-collectors)
5. [Les différents Garbage Collectors](#les-différents-garbage-collectors)
   1. [Cas d'utilisation](#cas-dutilisation)
   2. [Les principaux Garbage Collectors](#les-principaux-garbage-collectors)
6. [Conclusion - Comprendre les bases est essentiel](#conclusion---comprendre-les-bases-est-essentiel)

## Introduction

La gestion de la Heap Java fait peur.  
J’ai souvent entendu que c’était trop compliqué, que l’intérêt de Java est aussi de ne pas s’occuper de la gestion de la mémoire, ou encore que c’était inutile car quand l’application n’a pas assez de mémoire, il suffit de lui en affecter plus.

Mais pourquoi attribuer plus de ressources quand mieux configurer sa JVM peut suffire ? Devrait suffire.

Pour bien configurer sa JVM, encore faut-il avoir certaines connaissances de base sur la Heap Java ainsi que sur le processus qui la gère, le Garbage Collector (GC). Et encore faut-il savoir le monitorer afin de le configurer au mieux.

Dans cet article, je vous propose de découvrir de façon simplifiée le fonctionnement de la gestion de la Heap par le Garbage Collector, de connaître les différents algorithmes utilisés par les différents Garbage Collectors, et savoir choisir le Garbage Collector le plus adapté à ses besoins.

Malgré le titre, cet article n'est pas destiné aux "plus nuls" d'entre nous, il est destiné à tous... à vous les développeurs confirmés qui souhaitez enrichir vos connaissances de la Heap et des Garbage Collectors, à vous les développeurs qui souhaitez avoir une piqûre de rappel, et aux autres, vous les curieux, que vous fassiez partie de l'ingénierie logicielle ou pas.

J'ai essayé de concevoir cet article afin que chacun y trouve son compte. Ceux qui souhaitent seulement comprendre comment la Heap est gérée par le Garbage Collector pourront s'arrêter assez tôt. Ceux qui connaissent déjà, mais qui souhaitent comprendre en détail comment fonctionnent les plus utilisés des Garbage Collectors pourront lire cet article jusqu'au bout.

Avant de commencer à rentrer dans les détails, rappelons certaines définitions qui vous aideront à mieux appréhender la suite de cet article.

**Machine Virtuelle Java (ou JVM)**:

* Elle est un des éléments les plus importants de la plate-forme Java. Elle assure l'indépendance du matériel et du système d'exploitation lors de l'exécution des applications Java. Une application Java ne s'exécute pas directement dans le système d'exploitation mais dans une machine virtuelle, appelée JVM, qui s'exécute dans le système d'exploitation proposant ainsi une couche d'abstraction entre l'application Java et le système.

**La Heap Java :**

* Jean-Michel Doudoux en donne une définition simple et efficace dans son [blog](https://www.jmdoudoux.fr/java/dej/chap-jvm.htm#:~:text=Le%20tas%20(Heap),partag%C3%A9s%20par%20tous%20les%20threads.) : *"Cette zone de mémoire est partagée par tous les threads de la JVM : elle stocke toutes les instances des objets créés. Tous les objets créés sont obligatoirement stockés dans le tas (Heap) et sont donc partagés par tous les threads."*
* La heap est divisée en 2 zones principales, la "Young Generation" et la "Old Generation".
* La heap est une zone mémoire dans laquelle sont stockés vos objets et son nettoyage est géré par le "Ramasse-Miettes" (le Garbage Collector ou GC).

**Le Garbage Collector :**

* Il est responsable de vérifier l’état de la Heap afin de supprimer les objets qui ne sont plus utilisés / plus référencés (c’est-à-dire que plus aucun autre objet ne pointe sur ces derniers), et ainsi libérer de l’espace mémoire pour les nouveaux objets à créer.  
Il en existe de plusieurs types, chacun adapté à des besoins différents.

**Young Generation (également appelé Nursery) :**

* Les nouveaux objets créés par votre application sont placés dans cette zone mémoire. Mais nous verrons qu’elle ne contient pas uniquement les objets nouvellement créés.

**Old Generation (également appelé Tenured) :**

* Cet espace est utilisé pour stocker des objets qui survivent à des MinorGC. On dit alors que ces objets survivent longtemps.

**StopTheWorld :**

* Un "StopTheWorld", c’est très important à comprendre, équivaut à un freeze (arrêt) total des threads de votre application Java le temps que le Garbage Collector termine ce qu’il a à faire.

**MinorGC :**

* Quand le Garbage Collector estime que la "Young Generation" doit être nettoyée, il procède alors à un "MinorGC", déclenchant par ce fait ce qu’on appelle un "StopTheWorld". Les objets ayant survécu, sont déplacés dans la "Old Generation".

**MajorGC :**

* Quand le Garbage Collector estime que la Old Generation doit être nettoyée, il procède alors à un "MajorGC". Il déclenche également un "StopTheWorld", mais ce dernier peut s’avérer beaucoup plus lent car il implique des objets "vivants".

**FullGC :**

* Quand le Garbage Collector estime que la Old Generation **et** la Young Generation doivent être nettoyées, il procède alors à un "FullGC". Il déclenche donc également un "StopTheWorld".

## Le fonctionnement très simplifié de la Heap

Schématiquement, on pourrait représenter simplement la vie d’une Heap avec l’exemple suivant :

1. Initialisation de la Heap avec 3 nouveaux objets : 1, 2 et 3.  
![Step 1](/assets/2023/10/01/01-simple-explanation.png)

2. Les objets 1 et 2 sont déréférencés. L'objet 4 est créé.  
![Step 2](/assets/2023/10/01/02-simple-explanation.png)

3. La Young Generation est pleine. Les objets 5 et 6 sont en attente de création. Le Garbage Collector déclenche alors un MinorGC. Les objets déréférencés 1 et 2 sont supprimés. Les objets encore vivants (3 et 4) sont déplacés dans la Old Generation. Les nouveaux objets 5 et 6 sont créés/ajoutés dans la Young.  
![Step 3](/assets/2023/10/01/03-simple-explanation.png)

4. Les objets 5 et 6 sont déréférencés. Les nouveaux objets 7 et 8 sont créés. L'objet 3, présent dans la Old, est déréférencé.  
![Step 4](/assets/2023/10/01/04-simple-explanation.png)

5. La Young Generation est pleine. Le Garbage Collector déclenche un MinorGC. Les objets déréférencés 5 et 6 sont supprimés. Les objets encore vivants (7 et 8) sont déplacés dans la Old Generation.  
![Step 5](/assets/2023/10/01/05-simple-explanation.png)

6. L'objet 9 est créé. La Old Generation est pleine. Le Garbage Collector déclenche alors un MajorGC. L'objet déréférencé 3 est supprimé.  
![Step 6](/assets/2023/10/01/06-simple-explanation.png)

Au final, rien de compliqué, ça fonctionne de manière logique. Mais vous devez bien vous douter que ce n’est en réalité, pas si simple. Abordons le prochain chapitre pour en savoir plus.

## Le fonctionnement plus détaillé de la Heap

Il existe 2 zones mémoires principales (la Young et la Old), mais la Young est en fait elle-même découpée en 2 zones distinctes : l’Eden et le Survivor Space.

Le Survivor Space est lui-même également découpé en 2 zones : Survivor Space 0 (S0) et Survivor Space 1 (S1).

L’Eden est donc la zone mémoire dans laquelle sont stockés les nouveaux objets créés.  
Lors d’un MinorGC, les objets qui survivent sont déplacés dans une des deux zones Survivor, et leur âge est incrémenté.

Lors du MinorGC suivant, les objets présents dans un Survivor et qui ont survécu, sont déplacés dans l’autre Survivor. L’âge de ces derniers est alors incrémenté, jusqu’à un seuil. Une fois ce seuil atteint, les objets sont déplacés dans la Old Generation.

On va schématiser ce fonctionnement qui sera sans doute plus clair ensuite.

1. Initialisation de la Heap avec 3 nouveaux objets dans l'Eden : 1, 2 et 3.  
![Step 1](/assets/2023/10/01/07-detailed-explanation.png)

2. L'objet 3 est déréférencé.  
![Step 2](/assets/2023/10/01/08-detailed-explanation.png)

3. L'objet 4 est en attente de création. L'Eden étant plein, le Garbage Collector décide alors de déclencher un MinorGC. Les objets référencés 1 et 2 sont déplacés dans le Survivor Space 0 (S0) et leur âge est incrémenté de 1 (**numéro en rouge*). L'objet déréférencé 3 est supprimé. L'objet 4 est ajouté dans l'Eden.  
![Step 3](/assets/2023/10/01/09-detailed-explanation.png)

4. Les objets 5 et 6 sont créés. Les objets 2 et 4 sont déréférencés. L'objet 5 est déréférencé avant le passage du GC.  
![Step 4](/assets/2023/10/01/10-detailed-explanation.png)

5. L'Eden étant plein, le Garbage Collector décide de déclencher un Minor GC ayant pour conséquence de supprimer les objets 2, 4 et 5, et de déplacer les objets 1 et 6 dans le Survivor Space 1 (S1), tout en incrémentant leur âge. Les objets 7, 8 et 9 sont créés. Les objets 6 et 7 sont déréférencés.  
![Step 5](/assets/2023/10/01/11-detailed-explanation.png)

6. L'Eden étant plein, le Garbage Collector décide de déclencher un Minor GC ayant pour conséquence de supprimer les objets 6 et 7, de déplacer l'objet 1 dans le Survivor Space 0 tout en incrémentant son âge et de déplacer l'objet 8 dans le Survivor Space 0. Les objets 10 et 11 sont créés. Les objets 9 et 11 sont déréférencés.  
![Step 6](/assets/2023/10/01/12-detailed-explanation.png)

7. L'Eden étant plein, le Garbage Collector décide de déclencher un Minor GC ayant pour conséquence de supprimer les objets 9 et 11, de déplacer l'objet 1 (ayant atteint son âge max, ici 3) dans la Old Generation, et de déplacer les objets 8 et 10 dans le Survivor Space 1. Les objets 11, 12 et 13 sont déréférencés.  
![Step 7](/assets/2023/10/01/13-detailed-explanation.png)

8. *L'application continue de vivre, le Garbage Collector fait son travail et la Old Generation se remplit.*  
![Step N](/assets/2023/10/01/14-detailed-explanation.png)

9. La Old Generation étant pleine, le Garbage Collector décide de déclencher un MajorGC. Les objets 1 et 10 étant déréférencés, sont supprimés.  
![Step N+1](/assets/2023/10/01/15-detailed-explanation.png)

Jusqu’ici, ça reste assez simple à comprendre mais je suis persuadé que vous avez maintenant une question en tête : "*Pourquoi y-a-t-il 2 Survivor Spaces sachant qu'il y en a toujours un qui est vide ? On perd de l'espace mémoire...*"  
La réponse est assez simple en réalité : afin d'**éviter la fragmentation de la mémoire**. S'il n'y avait qu'un seul Survivor, lors d'un MinorGC, certains objets déréférencés et présents dans le Survivor seraient supprimés, laissant un espace vide dans la mémoire. Cet espace vide serait alors considéré comme de la fragmentation et il n'y aurait alors plus qu'une seule solution pour ne pas perdre cet espace : lancer une défragmentation. Et celle-ci ferait perdre du temps au Garbage Collector, c'est certain.  
En ayant 2 espaces Survivor distincts, les objets déplacés dans le Survivor vide, sont ajoutés de manière contigüe. Ainsi, aucune fragmentation n'est possible.

Maintenant que vous avez les notions de "Garbage Collection", voyons ce qu'il se cache derrière celles-ci.  
En effet, derrière le concept, il y a plusieurs grandes **phases** distinctes :

* **Mark** : étape consistant à parcourir le graph d’objets afin de marquer ceux encore référencés, en partant de la classe parente.
* **Sweep** : étape consistant en la suppression des objets non référencés.
* **Compact** : étape consistant à compacter/défragmenter la mémoire en déplaçant les objets de façon à les stocker de manière contigüe.
* **Copy** : étape consistant à copier les objets vivants, d'une région mémoire à une autre.

Et qui dit plusieurs Garbage Collectors, dit plusieurs algorithmes. C’est ce que nous verrons dans le prochain chapitre.

## Les algorithmes au sein des Garbage Collectors

En fonction du type de Garbage Collector choisi, un ou plusieurs algorithmes seront utilisés. On parle alors de **Collections**. Et chaque algorithme est basé sur plusieurs phases.

Voyons ces algorithmes plus en détail. Ils sont au nombre de 3.

* **Sweeping (basé sur les phases Mark-Sweep)**
  * Une fois la phase de marquage (*Mark*) terminée, tout l'espace occupé par des objets non référencés, est considéré comme libre et peut donc être réutilisé pour allouer de nouveaux objets.
  * Cette approche se base sur une liste de régions libres, ainsi que sur leur taille.
  * Le principal problème de cette approche est qu’il peut exister de nombreuses régions libres, mais si aucune région n'est suffisamment grande pour accueillir la nouvelle allocation, souvent causée par une grande fragmentation, celle-ci échouera en *OutOfMemoryError*.
* **Compacting (basé sur les phases Mark-Sweep-Compact)**
  * Cette approche résout les inconvénients de l’algorithme "*Mark-Sweep*" en déplaçant tous les objets marqués - et donc référencés - au début de la région mémoire.
  * L'inconvénient de cette approche est une augmentation de la durée du "StopTheWorld", car tous les objets doivent être copiés à un nouvel endroit et toutes les références à ces objets doivent être mises à jour.
  * Après l’opération de compactage, l'allocation d'un nouvel objet est à nouveau extrêmement rapide.
  * En utilisant cette approche, l'emplacement de l'espace libre est toujours connu et il n’y plus aucun problème de fragmentation.
* **Copying (basé sur les phases Mark-Copy)**
  * Très similaire à l’approche précédente, car cette approche "relocalise" tous les objets référencés.
  * Seule différence, c’est une région mémoire différente qui accueille les objets survivants. Mais la nécessité d’une région supplémentaire et le fait qu’elle doive être suffisamment grande pour accueillir les objets encore référencés sont aussi les principaux inconvénients.
  * La "*Copy*" peut se produire simultanément avec la phase de marquage ("*Mark*").

## Les différents Garbage Collectors

### Cas d'utilisation

Java fournit plusieurs Garbage Collectors, mais chacun est optimisé pour des cas d’utilisation différents et votre choix doit dépendre de ce que vous recherchez (Débit VS Latence VS Empreinte Mémoire).

Si votre application est transactionnelle, vous privilégierez un fort débit (ou Throughput) en impactant négativement la latence.

Si votre application doit avoir des temps de réponse très brefs (application affichant des pages Web par exemple), vous privilégierez une faible latence en impactant négativement le débit et l’empreinte mémoire.

En cherchant une façon d'imager "Latence VS Débit VS Empreinte mémoire", un concept que j’ai lu ou entendu, m’est revenu en tête.

*Depuis quelque temps déjà, l’écologie est un des sujets dont nous parlons le plus. Imaginons donc une usine de fabrication de batteries de voiture électrique, et posons l'hypothèse suivante : cette usine fabrique une batterie toutes les minutes, 24/7.*

*Mais, il faut 1 heure pour construire une batterie. Cette usine fabrique donc 60 batteries chaque heure.*  
*On peut en déduire que cette usine a :*

* *une latence de la chaîne de construction d'une batterie de 1 heure.*
* *un débit de 60 batteries par heure.*

*Lorsque les commandes de batteries sont stables, aucun problème de performance.*

*Imaginons maintenant que les commandes de batteries doublent, et qu’au lieu de produire 60\*24=1440 batteries par jour, il faille en produire 2880/jour. Les performances de cette usine ne suffiraient alors plus.*

*Naturellement, on serait tenté de dire que la latence, plutôt faible (1 heure), n’est pas un problème, et qu’il faudrait plutôt améliorer le débit en augmentant la capacité.*

*Pour ce faire, nous créerions alors une deuxième chaîne de production. Mais ceci a un coût et à chaque fois que l’entreprise devra augmenter son débit, il faudra qu’elle crée une nouvelle ligne de production.*

*Une alternative doit donc être envisagée pour régler ce problème de performance.*

*La latence considérée précédemment sans rapport avec le système cachait en fait une solution différente.*  
*Si celle-ci avait pu être réduite de 1 heure à 30 minutes, la même augmentation de débit aurait été possible sans aucune capacité de production supplémentaire.*

Ce que nous apprend ce concept est, que la réduction de la latence soit possible ou non, ou qu'elle soit économique, n'est pas pertinent dans ce cas. Ce qui importe, c'est un principe très similaire à celui que nous rencontrons tous les jours dans l'ingénierie logicielle : vous pouvez presque toujours choisir entre deux solutions à un problème de performances. Vous pouvez soit consacrer davantage de matériel au problème, soit passer du temps à choisir et à configurer correctement votre Garbage Collector (tout en optimisant votre code évidemment).

### Les principaux Garbage Collectors

Il existe plusieurs Garbage Collectors et vous allez voir qu'en réalité, ils sont basés sur plusieurs Collections d'algorithmes. En voici une liste non exhaustive :

* **Serial GC**
  * Il utilise l’algorithme "*Mark-Copy*" pour la Young et l’algorithme "*Mark-Sweep-Compact*" pour la Old.
  * Il s’exécute dans un seul thread sans parallélisation avec les Threads applicatifs.
  * Chaque étape est réalisée en "StopTheWorld".
  * Il privilégie donc l’empreinte mémoire la plus faible au détriment du débit et de la latence.
* **Parallel GC**
  * Il est aussi appelé "*Parallel Compaction*" Garbage Collector.
  * Il utilise l’algorithme "*Mark-Copy*" pour la Young et l’algorithme "*Mark-Sweep-Compact*" pour la Old.
  * Chaque étape est réalisée en "StopTheWorld".
  * Les phases de MinorGC et MajorGC sont exécutées par plusieurs threads en parallèle (d’où son nom).
  * Il privilégie le débit au détriment de la latence et de l’empreinte mémoire (parfait pour les traitements par lots).
  * Par défaut, le nombre de threads utilisés est égal au nombre de cores de la machine.
* **Concurrent GC (CMS)**
  * Il est aussi appelé "*Concurrent Mark and Sweep*" (CMS).
  * Il utilise l’algorithme "*Parallel Mark-Copy*" ("StopTheWorld") pour la Young et l’algorithme "*Concurrent Mark-Sweep*" pour la Old.
  * Il est conçu pour éviter les longues pauses lors des collectes dans la Old Generation : pas de compactage, mais gestion d’une liste d’espaces libres.
  * Il privilégie la latence la plus faible au détriment du débit et de l’empreinte mémoire.
  * Par défaut, il utilise 1/4 du nombre de cores physiques de la machine.
  * Vous retrouverez son fonctionnement détaillé dans un futur article.
* **Garbage First Garbage Collector (G1GC)** :
  * Son fonctionnement étant bien différent des autres, je vous propose d'en parler dans un futur article.

## Conclusion - Comprendre les bases est essentiel

Le but premier de cet article est de permettre au plus grand nombre de comprendre, voire même de découvrir, la notion de *Garbage Collections*.  
Plus vous avancez dans l'article, plus vous avez de détails. J'ai lu beaucoup d'articles sur ce sujet au début et tout au long de ma carrière (encore maintenant), mais je les ai toujours trouvés soit trop compliqués soit pas assez détaillés pour des débutants.  
J'espère que celui-là permettra de démystifier la gestion de la mémoire dans le langage Java et qu'il permettre également à des passionnés comme moi d'appréhender les GC en toute tranquilité.  

Pour écrire cet article, je me suis beaucoup appuyé sur mon expérience et mes nombreuses lectures de GCLogs, mais aussi et principalement sur la documentation d'Oracle : [le SerialGC, le ParallelGC et le CMS](https://docs.oracle.com/javase/9/gctuning/toc.htm).  

On se retrouve bientôt pour un nouvel article sur les Garbage Collectors, dans lequel on verra plus en détail le fonctionnement des deux principaux : le CMS et le G1GC.
