---
layout: post
title: "OpenRewrite, pour les migrer tous… et dans le refactoring les lier"
description: "Le sujet de cet article est de présenter OpenRewrite, un outils permettant via les plugins Maven ou 
Gradle, ou via le CLI, d'upgrader techniquement votre code. Migrer de Spring Boot 2 à Spring Boot 3, migrer de 
JUnit4 à JUnit5, migrer vers Java 21... Vous y retrouverez également des exemples de code disponibles sur mon répo 
Github."
resume: "Le sujet de cet article est de présenter OpenRewrite, un outils permettant via les plugins Maven ou
Gradle, ou via le CLI, d'upgrader techniquement votre code. Migrer de Spring Boot 2 à Spring Boot 3, migrer de
JUnit4 à JUnit5, migrer vers Java 21... Vous y retrouverez également des exemples de code disponibles sur mon répo 
Github."
image: "/assets/images/openrewrite.jpg"
readTime: "25"
excerpt: OpenRewrite Moderne refactoring Java Spring Boot Migration JUnit4 JUnit5 Java 21 recettes recipes
---

## Table des matières <!-- omit in toc -->

1. [Introduction](#introduction)
2. [OpenRewrite ?](#openrewrite-)
3. [Comment ça marche ?](#comment-ça-marche-)
4. [Intégration par l'exemple](#intégration-par-lexemple)
    1. [Migrer avec OpenRewrite](#migrer-avec-openrewrite)
    2. [Ajouter la prise en compte des Threads Virtuels](#ajouter-la-prise-en-compte-des-threads-virtuels)
    3. [Dans les profondeurs du code fut forgé… un plugin OpenRewrite 🔥](#dans-les-profondeurs-du-code-fut-forgé-un-plugin-openrewrite-)
    4. [Les résultats](#les-résultats)
5. [Allez plus loin dans la Terre du Milieu 🧙‍♂️](#allez-plus-loin-dans-la-terre-du-milieu-)
6. [OpenRewrite, un outil pour les migrer tous 💍](#openrewrite-un-outil-pour-les-migrer-tous-)
7. [📜 Liens Utiles](#-liens-utiles)
8. [🔥 Notre code](#-notre-code)

## Introduction

Migrer de Java 8 à Java 21 vous fait peur ? Migrer de Spring Boot 2 à Spring Boot 3 vous angoisse ? Migrer de JUnit 4 à JUnit 5 coûte trop cher ?

Et si je vous disais qu’un outil pouvait automatiser toutes ces migrations — de Java EE vers Jakarta EE, des anciennes API *Date/Calendar* vers *java.time*, et bien plus encore — sans prise de tête ni perte de temps ?

Vous en rêviez, **OpenRewrite** le fait ! 🎉

## OpenRewrite ?

*OpenRewrite est un écosystème Open-Source de re-factorisation automatisée permettant aux développeurs d'éliminer efficacement la dette technique au sein de leurs référentiels.*

Sa promesse ? Réduire le temps des upgrades techniques de plusieurs jours à quelques minutes. Dans l'esprit, ça interpelle, non ?

## Comment ça marche ?

OpenRewrite fonctionne en modifiant ce qu'ils appellent des arbres sémantiques sans perte ([LST](https://docs.openrewrite.org/concepts-and-explanations/lossless-semantic-trees)) représentant votre code source, puis en modifiant ces arbres et en les réintégrant dans votre code.  
Pour vulgariser, LST est une représentation arborescente de votre code.  

Le concept vous parait compliqué ? Voyez le LST comme un AST (Abstract Syntax Tree) mais enrichi : avec les attributs de type et le format (espaces avant/après etc) qui permet un rendu, après transformation de votre code, identique à l'origine. Les modifications sur les LST sont réalisées dans un ou plusieurs visiteurs (*Pattern Visitor*), eux-mêmes regroupés en recettes (*recipes*).

Voici une illustration assez simple que propose OpenRewrite sur son site :

![LST](/assets/2025/07/09/LST.png)

Principalement conçu pour Java, OpenRewrite est capable de traiter d'autres langages comme Kotlin, Groovy, XML, YAML, JSON, SQL... Mais pour la suite, nous nous focaliserons principalement sur Java (avec un soupçon de YAML et de XML).

OK, c'est bien beau tout ça, mais comment je l'intègre dans mon projet et comment je dois m'en servir ?

## Intégration par l'exemple

Pour bien comprendre ce que vous apporte OpenRewrite, nous allons l'intégrer dans un projet Java ([disponible ici](https://github.com/cedricSarre/open-rewrite-sample)) basé sur les technonolgies suivantes, que vous retrouverez dans la branche *main* :

* Java 8
* Spring Boot 2
* Junit 4
* Mockito 3
* Maven

Notre but ici est de migrer vers :

* Java 21
* Spring Boot 3.3
* Junit 5
* Mockito 5

### Migrer avec OpenRewrite

OpenRewrite propose de réaliser ces migrations soit via un CLI (*rewrite CLI*), soit via les plugins Maven (*rewrite-maven-plugin*) ou Gradle (*org.openrewrite.rewrite*). Il existe également une solution payante appelée "Moderne" que nous n'aborderons pas. Nous utiliserons par la suite le plugin Maven.

Pour ce faire, rien de plus simple, ajoutons ce dernier à notre fichier *pom.xml* :

```xml
    <plugin>
        <groupId>org.openrewrite.maven</groupId>
        <artifactId>rewrite-maven-plugin</artifactId>
    </plugin>
```

Rappelez-vous que nous souhaitons migrer vers Java 21, Spring Boot 3.3, JUnit 5 et Mockito 5. Pour celà, il suffit de récupérer sur le site d'OpenRewrite la configuration des recettes équivalentes :

* [Java 21](https://docs.openrewrite.org/running-recipes/popular-recipe-guides/migrate-to-java-21#example-configuration)
* [Spring Boot 3.3](https://docs.openrewrite.org/recipes/java/spring/boot3/upgradespringboot_3_3#usage) : cette dernière inclut la migration vers JUnit5.
* [Mockito 5](https://docs.openrewrite.org/recipes/java/testing/mockito/mockito1to5migration#usage)

> **Note**  
> En cherchant une recette spécifique sur le site d'OpenRewrite, vous constaterez plusieurs mentions dans le chapitre "*Recipe Source*" :
>
> * *This recipe is available under the Moderne Source Available License.* (en français, *Cette recette est disponible sous la licence Moderne Source Available.*)
> * *This recipe is only available to users of Moderne.* (en français, *Cette recette est uniquement accessible aux utilisateurs de Moderne.*)
>
> Et là, ça change tout.  
> En effet, la première vous autorise à utiliser la recette sans la modifier, sans la redistribuer et sans l'intègrer dans un produit ou une offre commerciale.  
> Tandis que la deuxième ne vous permet ni ne vous autorise à l'utiliser.

Comment intégrer les recettes dans la configuration du plugin ? Rien de plus simple, une fois votre liste établie, il suffit d'ajouter ce bloc dans votre déclaration du plugin Maven :

```xml
<configuration>
    <activeRecipes>
        <recipe>org.openrewrite.java.migrate.UpgradeToJava21</recipe>
        <recipe>org.openrewrite.java.spring.boot3.UpgradeSpringBoot_3_3</recipe>
        <recipe>org.openrewrite.java.testing.mockito.Mockito1to5Migration</recipe>
    </activeRecipes>
</configuration>
```

Certaines recettes ont également besoin de dépendances Maven supplémentaires. Vous trouverez en détail lesquelles dans chacune des recettes depuis le site OpenRewrite. Ces dépendances sont à ajouter dans votre déclaration du plugin.

La recette de migration Java 21 a besoin de la dépendance :

```xml
<dependency>
    <groupId>org.openrewrite.recipe</groupId>
    <artifactId>rewrite-migrate-java</artifactId>
</dependency>
```

La recette Spring Boot a besoin de la dépendance :

```xml
<dependency>
    <groupId>org.openrewrite.recipe</groupId>
    <artifactId>rewrite-spring</artifactId>
</dependency>
```

Enfin, la recette Mockito 5 a besoin de celle-ci :

```xml
<dependency>
    <groupId>org.openrewrite.recipe</groupId>
    <artifactId>rewrite-testing-frameworks</artifactId>
</dependency>
```

Une fois cette configuration - honnêtement très rapide - réalisée, vous n'avez plus qu'à exécuter le plugin OpenRewrite via la commande `mvn rewrite:run`.

Analysons ensuite les logs du build pour la partie migration vers Java 21 :

```log
[INFO] Using active recipe(s) [org.openrewrite.java.migrate.UpgradeToJava21, org.openrewrite.java.spring.boot3.UpgradeSpringBoot_3_3, org.openrewrite.java.testing.mockito.Mockito1to5Migration]
...
[WARNING] Changes have been made to pom.xml by:
[WARNING]     org.openrewrite.java.migrate.UpgradeToJava21
[WARNING]         org.openrewrite.java.migrate.UpgradeToJava17
[WARNING]             org.openrewrite.java.migrate.Java8toJava11
[WARNING]                 org.openrewrite.java.migrate.UpgradeBuildToJava11
[WARNING]                     org.openrewrite.java.migrate.UpgradeJavaVersion: {version=11}
[WARNING]                         org.openrewrite.maven.UpdateMavenProjectPropertyJavaVersion: {version=11}
[WARNING]             org.openrewrite.java.migrate.UpgradeBuildToJava17
[WARNING]                 org.openrewrite.java.migrate.UpgradeJavaVersion: {version=17}
[WARNING]                     org.openrewrite.maven.UpdateMavenProjectPropertyJavaVersion: {version=17}
[WARNING]             org.openrewrite.java.migrate.UpgradePluginsForJava17
[WARNING]                 org.openrewrite.maven.UpgradePluginVersion: {groupId=org.apache.maven.plugins, artifactId=maven-compiler-plugin, newVersion=3.x}
[WARNING]             org.openrewrite.java.migrate.AddLombokMapstructBinding
[WARNING]                 org.openrewrite.maven.AddDependency: {groupId=org.projectlombok, artifactId=lombok-mapstruct-binding, version=0.2.0, acceptTransitive=false}
[WARNING]         org.openrewrite.java.migrate.UpgradeBuildToJava21
[WARNING]             org.openrewrite.java.migrate.UpgradeJavaVersion: {version=21}
[WARNING]                 org.openrewrite.maven.UpdateMavenProjectPropertyJavaVersion: {version=21}
[WARNING] Changes have been made to src\main\java\fr\example\sample_open_rewrite\classroom\ClassroomController.java by:
[WARNING]     org.openrewrite.java.migrate.UpgradeToJava21
[WARNING]         org.openrewrite.java.migrate.UpgradeToJava17
[WARNING]             org.openrewrite.java.migrate.Java8toJava11
[WARNING]                 org.openrewrite.java.migrate.UpgradeBuildToJava11
[WARNING]                     org.openrewrite.java.migrate.UpgradeJavaVersion: {version=11}
[WARNING]             org.openrewrite.java.migrate.UpgradeBuildToJava17
[WARNING]                 org.openrewrite.java.migrate.UpgradeJavaVersion: {version=17}
[WARNING]         org.openrewrite.java.migrate.UpgradeBuildToJava21
[WARNING]             org.openrewrite.java.migrate.UpgradeJavaVersion: {version=21}
...
```

On remarque tout de suite que OpenRewrite procède par étape, migration de Java 8 vers Java 11, puis vers Java 17 et enfin vers Java 21, dans un premier temps dans le fichier `pom.xml`, puis dans chaque classe.

En analysant le log dans son intégralité, on remarque qu'il fait la même chose pour chaque recette.

Enfin, Le log se termine par :

```log
[WARNING] Please review and commit the results.
[WARNING] Estimate time saved: 10m
```

OpenRewrite a estimé que les changements qu'il a réalisés ont permis d'économiser 10 minutes de développement. Pour l'avoir fait manuellement, il m'a clairement fallu plus de temps. Mais cette application est très petite, donc on peut le croire.

Il nous informe aussi que les changements sont disponibles directement dans notre code.  
Si nous jetons un œil aux classes Java, on remarque qu'il a modifié les imports `import javax.xxx` en `import jakarta.xxx`. On en déduit facilement que cette modification est dûe à la migration de Java 8 à 21.

OpenRewrite via la recette de migration Spring Boot a également modifié les propriétés dans les fichiers `application.yml`, en modifiant la propriété `management.metrics.export.prometheus.enabled: false` en `management.prometheus.metrics.export.enabled: false`.

Si on jette un œil au `pom.xml`, on remarque quelques changements majeurs :

* la version du starter `spring-boot-starter-parent` est passée de *2.7.X* à *3.3.X*
* les propriétés `maven.compiler.source`, `maven.compiler.target` et `java.version` sont passées de *8* à *21*
* la dépendance `springdoc-openapi-ui` a été migrée en `springdoc-openapi-starter-webmvc-ui`
* la dépendance `lombok-mapstruct-binding` a été ajoutée
* la dépendance `junit-vintage-engine` a été supprimée
* la version de la dépendance `mockito-core` a été retirée, Mockito 5 étant tiré transitivement par *Spring Boot Test 3* (elle aurait même pu être retirée...)

Plus qu'à builder le projet et surprise, ça compile, ça builde, les tests passent, la migration est terminée 🥳

> **💡Tips**  
> [Retrouvez le commit de la configuration OpenRewrite](https://github.com/cedricSarre/open-rewrite-sample/commit/c08216f46a16c989f58df68cae85358f1cdb69b1)

### Ajouter la prise en compte des Threads Virtuels

Java 21 et Spring Boot 3 gèrent les Threads Virtuels. Donc autant les activer.  
Est-ce qu'OpenRewrite le permet ? Oui !! et non...  
Disons qu'il n'y a pas de recette toute faite pour ajouter cette propriété, mais ça reste très simple à faire !

Créeons notre propre recette en nous basant sur une déjà existante ! On commence par créer le fichier `.rewrite/add-virtual-threads-spring-property.yml`, à la racine de notre projet, contenant :

```yml
# type: Informe OpenRewrite que ce fichier contient une recette, conforme au schéma v1beta publié par OpenRewrite.
type: specs.openrewrite.org/v1beta/recipe

# name: Identifiant interne (sans espace, en camel case) de notre recette
name: AddSpringVirtualThreadsProperty 

# displayName : Nom visible dans Moderne ou dans le plugin Intellij "Rewrite Intellij Plugin"
displayName: Add `spring.threads.virtual.enabled=true` property to the application.yml/properties file

# Liste des recettes à appliquer
recipeList:

    # Recette incluse dans OpenRewrite, fournie par le module rewrite-spring, elle sert à ajouter une propriété Spring
  - org.openrewrite.java.spring.AddSpringProperty: 

        # property : Propriété Spring à ajouter
        property: spring.threads.virtual.enabled 

        # value : Valeur que prendra la propriété Spring ajoutée
        value: "true" 
```

Côté `pom.xml` et configuration du plugin, il suffit d'ajouter notre recette (dans la balise `<activeRecipes>`)

```xml
<recipe>AddSpringVirtualThreadsProperty</recipe>
```

et l'emplacement de celle-ci (dans la balise `<configuration>`) :

```xml
<configLocation>.rewrite/add-virtual-threads-spring-property.yml</configLocation>
```

> **💡Tips**  
> [Retrouvez le commit associé](https://github.com/cedricSarre/open-rewrite-sample/commit/80d992497ba799797d60e0995157ee29c943c0bd)

Plus qu'à relancer la commande `mvn rewrite:run` et à jeter un œil aux logs et aux modifications :

```log
[WARNING] Changes have been made to src\main\resources\application.yml by:
[WARNING]     AddSpringVirtualThreadsProperty
[WARNING]         org.openrewrite.java.spring.AddSpringProperty: {property=spring.threads.virtual.enabled, value=true}
[WARNING] Please review and commit the results.
[WARNING] Estimate time saved: 5m
```

Côté `application.yml`, la propriété est bien apparue 🥳

**Bilan**, en ajoutant simplement le plugin OpenRewrite à notre `pom.xml`, en lui configurant 3 recettes existantes (faciles à trouver dans la documentation), en lui ajoutant les dépendances requises, et en créant notre propre recette pour ajouter une propriété Spring, on a migré notre application legacy vers une application Spring Boot 3, Java 21, avec les dernières versions des frameworks de tests, on a migré les propriétés Spring Boot et on a activé les Threads Virtuels ! Tout ça en 10 minutes ? 15 maximum ? Quel bonheur 😍

> **💡Tips**  
> [Retrouvez les résultats de la migration](https://github.com/cedricSarre/open-rewrite-sample/commit/930bdd580ed2fdd149303f5e8d84fa56a547c90c)

### Dans les profondeurs du code fut forgé… un plugin OpenRewrite 🔥

Je vous entends déjà dire "*c'est tout ce que sait faire OpenRewrite ?*"  
Evidemment que non 😁 OpenRewrite permet d'écrire ses propres recettes, ou devrais-je plutôt dire qu'il permet de **coder** ses propres recettes !  

Voyons comment développer une recette qui ajouterait l'annotation `@Tag("unit")` sur nos tests unitaires et l'annotation `@Tag("integration")` sur nos tests d'intégration. Tags qui nous permettront de configurer plus simplement les plugins Maven *Surefire* et *FailSafe*, respectivement comme suit :

```xml
<configuration>
  <groups>unit</groups>
</configuration>
```

et

```xml
<configuration>
  <groups>integration</groups>
</configuration>
```

Première chose à faire, écrire notre `pom.xml`. On va avoir besoin des dépendances `rewrite-java-21`, `rewrite-test`, `rewrite-testing-frameworks` et de `junit-bom`.

A première vue, coder une recette est assez simple, on doit étendre la classe `Recipe` et implémenter quelques méthodes : `getDisplayName()`, `getDescription()` et `getVisitor()`.  
Les deux premières, vous l'aurez compris, sont simples.  
La dernière en revanche doit contenir notre algorithme. Celle-ci va devoir retourner un `TreeVisitor<?, ExecutionContext>` — ici, une instance d'une classe anonyme qui hérite de `JavaIsoVisitor<>`, et il faut choisir les bonnes méthodes à implémenter en fonction de notre use-case. Nous savons déjà que nous souhaitons ajouter une annotation à chacune de nos classes de tests et y ajouter l'import qui convient. Nous allons donc implémenter les méthodes `visitCompilationUnit()` et `visitClassDeclaration`.

La méthode `visitCompilationUnit()` est appelée au tout début de la visite d'un fichier Java. C'est au sein de celle-ci que nous pouvons analyser et/ou modifier les imports, les annotations de classes, les packages...

La méthode `visitClassDeclaration` est appelée à chaque fois qu'une classe est rencontrée dans le code. Elle permet de vérifier si une classe est un test, ajouter/supprimer une annotation de classe, renommer la classe, inspecter les méthodes et les champs...

En d’autres termes, la première vous fait entrer dans Khazad-dûm, le vaste royaume des Nains, et la seconde vous guide à travers ses grandes salles, où reposent les secrets enfouis du code : les classes.

Mais pour commencer notre quête, il nous faut d’abord identifier les classes qui nous intéressent : les classes de tests. Facile, en général elles se trouvent dans `src/test` et suivent le pattern `*\*Test.java*`. On s'assure également qu'elles contiennent au moins une méthode annotée `@Test`.

Ensuite, on va différencier les classes de tests d'intégration des classes de tests unitaires. La méthode que nous avons choisie repose sur la recherche d'une annotation de la classe elle-même : si celle-ci est annotée par une des annotations suivantes, on en conclut que c'est une classe de tests d'intégration : `@SpringBootTest`, `@DataJpaTest`, `@WebMvcTest`, `@WebFluxTest`, `@JdbcTest`, `@DataMongoTest`, `@DataRedisTest`, `@DataCassandraTest`, `@RestClientTest`. Cette liste n'est sans doute pas exhaustive, mais dans notre cas, on prendra comme hypothèse que c'est suffisant.

On vérifie également que cette classe ne contient pas déjà l'annotation `@Tag` et l'import associé `org.junit.jupiter.api.Tag`. Ensuite, on ajoute l'annotation `@Tag("unit")` ou `@Tag("integration")` et l'import.

Notre recette est terminée, mais nous vous conseillons de réaliser une dernière chose importante, ajouter des tests unitaires pour valider celle-ci.

On compile, on génère le jar et en route pour l'intégration dans notre projet !

Là, encore rien de plus simple, ajoutez la recette à la liste déjà présente

```xml
<activeRecipes>
    <recipe>fr.example.recipe.AddTestTagRecipe</recipe>
</activeRecipes>
```

et la dépendance vers la recette

```xml
<dependency>
    <groupId>fr.example</groupId>
    <artifactId>add-test-tag-open-rewrite-recipe</artifactId>
    <version>1.0.0</version>
</dependency>
```

Une chose à savoir cependant. Votre recette utilise des éléments de JUnit5, il faut donc que votre déclaration de plugin OpenRewrite contienne la dépendance vers JUnit5.

Relancez la commande `mvn rewrite:run` et... votre application est upgradée, et tous vos tests sont annotés 😍

> **💡Tips**  
> [Retrouvez le code de cette recette](https://github.com/cedricSarre/add-test-tag-open-rewrite-recipe)  
> [Retrouvez l'intégration de cette recette dans notre application](https://github.com/cedricSarre/open-rewrite-sample/commit/83242ed62bcc56fc791c896abb79fd4d7a3ba17f)

### Les résultats

Les comparaisons ci-dessous montrent le travail réalisé par OpenRewrite en appliquant l'ensemble des recettes que nous avons vues.

* Exemple de comparaison du **code source** :
    ![Code source comparaison](/assets/2025/07/09/result0.png)

* Exemples de comparaison des imports et annotations d'une **classe de tests unitaires** :
    ![Test unitaire comparaison](/assets/2025/07/09/result1-0.png)  

* Exemples de comparaison de code d'une **classe de tests unitaires** :
    ![Test unitaire comparaison](/assets/2025/07/09/result1-1.png)

* Exemple de comparaison des imports et annotations d'une **classe de tests d'intégration** :
    ![Test d'intégration comparaison](/assets/2025/07/09/result2-0.png)

* Exemple de comparaison de code d'une **classe de tests d'intégration** :
    ![Test d'intégration comparaison](/assets/2025/07/09/result2-1.png)

* Exemples de comparaison du **pom.xml** :
    ![Pom.xml comparaison](/assets/2025/07/09/result3-0.png)

    ![Pom.xml comparaison](/assets/2025/07/09/result3-1.png)

* Comparaison du **fichier de configuration** :
    ![Configuration comparaison](/assets/2025/07/09/result4.png)

> **💡Tips**  
> [Retrouvez le résultat de la migration](https://github.com/cedricSarre/open-rewrite-sample/commit/e8140bd3dc9a73255e2873d834b1e3546a024fc9)

## Allez plus loin dans la Terre du Milieu 🧙‍♂️

L’aventure ne s’arrête pas là ! Pour explorer d’autres contrées et perfectionner votre maîtrise d’OpenRewrite :

* parcourez la documentation officielle, vous y découvrirez d'autres recettes et cas d'usage,
* testez l'intégration avec Gradle ou le CLI pour des projets non Maven ou Gradle,
* expérimentez la création de vos propres recettes...

## OpenRewrite, un outil pour les migrer tous 💍

Vous l'aurez compris, **OpenRewrite** ne demande qu'une courte montée en compétence, et très vite, vous serez capable d'upgrader vos applications en un rien de temps.  
Netflix l'a utilisé pour migrer ses quelques 3000 applications Spring Boot, pourquoi pas vous ?

Certes, les recettes les plus récentes nécessitent une licence, mais comme vous l'avez vu, on peut déjà faire énormément - rapidement et simplement - avec celles fournies gratuitement.

Mon conseil ? Essayez-le ! Vous ne verrez plus vos migrations techniques du même œil 😉

## 📜 Liens Utiles

* [Site Officiel](https://openrewrite.org/)  
* [AST](https://fr.wikipedia.org/wiki/Arbre_de_la_syntaxe_abstraite)  
* [Pattern Visitor](https://fr.wikipedia.org/wiki/Visiteur_(patron_de_conception))
* [Communauté Open Rewrite sur GitHub](https://github.com/orgs/openrewrite/repositories)

## 🔥 Notre code

* [Application Java](https://github.com/cedricSarre/open-rewrite-sample/tree/main)  
* [Plugin OpenRewrite](https://github.com/cedricSarre/add-test-tag-open-rewrite-recipe)
