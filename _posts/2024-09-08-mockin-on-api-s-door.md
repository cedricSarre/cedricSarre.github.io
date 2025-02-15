---
layout: post
title: "Mockin' On API's Door"
description: "Le sujet de cet article est de présenter trois des principales solutions pour mocker vos API REST : WireMock, MockServer et OpenAPI Mock. Lequel a le plus de fonctionnalités ? Lequel est le plus simple à mettre en place et à utiliser ? Lequel tire son épingle du jeu ?"
resume: "Le sujet de cet article est de présenter trois des principales solutions pour mocker vos API REST : WireMock, 
MockServer et OpenAPI Mock. Lequel a le plus de fonctionnalités ? Lequel est le plus simple à mettre en place et à utiliser ? Lequel tire son épingle du jeu ?"
image: "/assets/images/mocks.webp"
readTime: "15"
excerpt: mock REST Wiremock MockServer OpenAPI java
---

## Table des matières <!-- omit in toc -->

1. [Introduction](#introduction)
2. [Intérêts du mock](#intérêts-du-mock)
3. [Les solutions pour une application Java](#les-solutions-pour-une-application-java)
4. [Une application de test](#une-application-de-test)
  1. [MockServer](#mockserver)
     1. [Vous êtes restés sur votre faim ?](#vous-êtes-restés-sur-votre-faim-)
  2. [Wiremock](#wiremock)
     1. [Encore faim ?](#encore-faim-)
  4. [OpenAPI Mock](#openapi-mock)
5. [A vous de choisir](#a-vous-de-choisir)

## Introduction

Les tests sont importants dans la vie d'un logiciel.

Nombreuses sont les applications qui aujourd'hui, réalisent des appels HTTP(S) vers d'autres applications, qu'elles soient externes au Système d'Information ou internes. Il est généralement assez difficile de tester ces appels, et en général la solution priviligiée est l'utilisation de bouchon (ou mock).

Chacun y va de sa propre solution : écrire son propre mock, utiliser une instance de mock sur étagère (plugin, librairie) ou mocker directement la couche Service réalisant les appels HTTP.

Développer son propre mock a le bénéfice de pouvoir facilement le maintenir et le faire évoluer pour coller au plus près des besoins. Mais ça reste vrai seulement tant que l'équipe de développement garde la connaissance de ce dernier. Il comporte également certains inconvénients, comme gérer son déploiement ou encore le besoin de le considérer comme une application à part entière (tests, qualité, maintenance...).

Mocker les services via l'utilisation de librairies telles que Mockito, est une bonne solution au sein des tests unitaires pour isoler une unité de code (classe, méthode) et afin de tester son comportement en dehors de son environnement réel. Mais ce genre de solutions devient beaucoup moins intéressante dans le cadre des tests d'intégration, lors desquels nous souhaiterons tester également l'interaction entre différentes briques logicielles : base de données, gestionnaire d'identité, application logicielle externe...

Intéressons-nous donc aux tests d'intégration, aux solutions existantes de mocks pour Java - et plus particulièrement de mocks d'API HTTP - et comparons-les pour nous donner la capacité de bien choisir la solution la mieux adaptée à un besoin.

Le but n'est pas ici de faire un descriptif complet des fonctionnalités de chacune des solutions de mock que nous allons présenter, ni de lister toutes les solutions pour mocker une API HTTP, mais de découvrir quelles sont les solutions existantes et de se faire une idée de leurs points forts.

## Intérêts du mock

Les intérêts d'utiliser un mock pour simuler une API HTTP sont nombreux.

* En utilisant un mock d'API, l'équipe de développement peut travailler de manière indépendante, sans avoir à attendre que l'API réelle soit finalisée et/ou disponible. Ce qui peut accélérer le processus de développement.

* En simulant le comportement d'une API réelle, nous pouvons reproduire des scénarios spécifiques pour le débogage, pour les tests, ce qui facilite l'identification et la correction des erreurs dès le début du processus de développement.

* Un mock permet de développer et de tester une application même sans accès à l'API, ce qui est particulièrement utile dans les environnements de développements où l'accès à une API peut être inexistant.

* Un des intérêts majeurs de l'utilisation d'un mock est d'avoir le contrôle total sur les réponses de l'API simulée, ce qui permet de simuler des cas d'utilisation spécifiques ou des erreurs pour tester la réaction de l'application dans différentes situations.

* Enfin, utiliser un mock pendant le développement permet de réduire les coûts associés à l'utilisation de l'API réelle, surtout si celle-ci implique des frais d'utilisation ou de transaction.

## Les solutions pour une application Java

Les solutions sont (très) nombreuses, comme nous avons pu le mentionner lors du talk "[API, tu te mock de moi"](https://www.younup.fr/blog/replay-meet-up-api-tu-te-mock-de-moi), nous ne pourrons donc pas toutes les comparer mais nous nous attarderons ici sur trois solutions dites _"As Code"_ : __MockServer__, __Wiremock__ et __OpenApi Mock__.

Aujourd'hui, Docker est partout, et _Testcontainers_ est en train de s'imposer comme __LE__ framework de "_test dependencies as code_". Nous prendrons donc comme hypothèse que nos tests utilisent cette solution. Nous prendrons également comme hypothèse que notre application est basée sur Spring Boot (en version 3.X).

## Une application de test

Vous trouverez [ici](https://github.com/Younup/MockinOnApiSDoor) une application Spring Boot qui contient trois tests d'intégration correspondant à chacune des solutions de mock que nous allons aborder.

Cette application de gestion de tâches appelle un service météo externe pour enrichir l'expérience de l'utilisateur en lui fournissant des informations météorologiques pertinentes pour chaque tâche qu'il crée. Cela permet à l'utilisateur de mieux planifier et d'adapter ses activités en fonction des conditions météorologiques attendues.

### MockServer

_MockServer_ est un des outils de simulation d'API HTTP(S).

Il peut retourner des réponses spécifiques en fonction des requêtes qu'il reçoit. Il peut faire office de proxy en modifiant potentiellement les requêtes et/ou les réponses. Il peut également faire les deux : proxy pour certaines requêtes et mock pour d'autres.

Nous nous focaliserons ici sur la fonctionnalité de mock.

Pour cela, il se base sur la configuration d'_expectations_ (attentes) afin de connaître quelle(s) action(s) il doit effectuer en fonction des requêtes qu'il reçoit.  
Pour créer une _expectation_, il faut définir un _request matcher_ et la réponse qui doit être retournée.  
Lorsque _MockServer_ reçoit une requête, il va alors rechercher l'_expectation_ qui correspond et retourner la réponse attendue.

Plusieurs façons de l'utiliser sont possibles : via le "_Client API_", via le plugin Maven, en tant que conteneur Docker seul, via JUnit (en versions 4 et 5), en tant que _TestExecutionListener_ (Spring), ou encore via TestContainer en tant que conteneur Docker.

Pour utiliser _MockServer_ via _Testcontainers_, rien de plus simple, il suffit de suivre les étapes suivantes :

* Ajout des dépendances `org.mock-server:mockserver-netty` et `org.testcontainers:mock-server`
* Annotation de la classe de test avec `@Testcontainers`
* Déclaration d'un container "_mockServer_" dans la classe de test :

  ```java
    @Testcontainers
    class MockerServerIntegrationTest{
      
      @Container
      private static final MockServerContainer mockServerContainer = new MockServerContainer(DockerImageName.parse("mockserver/mockserver:5.15.0"));
      
      [...]
    }
  ```

* Création d'un `MockServerClient`
* Envoi de l'_expectation_ au _MockServer_ (au sein de la méthode de test par exemple) qui retourne un tableau d'`Expectations`:

  ```java
    [...]

    // Expectation
    Expectation[] expectations = mockServerClient.when(
        request()
          .withMethod("GET")
          .withPath("/my_path")
    )
    .respond(
        response()
          .withStatusCode(200)
          .withBody("...")
    );

    [...]
  ```

* Enfin, il suffit de requêter notre application pour vérifier qu'elle utilise bien le mock défini et que la réponse correspond à celle attendue.

  ```java
    [...]

    given()
      .contentType(ContentType.JSON)
      .when()
      .get("/myApplicationPath")
      .then()
      .statusCode(200)
      .body("...");
  ```

_MockServer_ peut également utiliser une [spécification OpenAPI v3](https://www.mock-server.com/mock_server/using_openapi.html) via les paramètres _"specUrlOrPayload"_ permettant de définir où trouver la spécification et _"operationsAndResponses"_ permettant entre autres choses de définir le statut de la réponse en fonction de l'_operationId_. Par exemple :

```json
{
  "specUrlOrPayload": "/my_path/my_spec.yaml",
  "operationsAndResponses": {
      "getMyPath": "200",
      "deleteMyPath": "403"
  }
}
```

Notre exemple d'utilisation de _MockServer_ est disponible [ici](https://github.com/Younup/MockinOnApiSDoor/blob/main/src/test/java/fr/younup/mock/MockServerIntegrationTest.java).

#### Vous êtes restés sur votre faim ?

Vous avez un doute sur le fait que votre application a bien fait appel à _MockServer_ ? Don't panic, il vous permet de vérifier vos appels et le nombre de fois qu'il a reçu une requête (nombre exact, minimum, maximum, jamais...) via la commande :

```java
  mockServerClient.verify(expectations[0].getId(), VerificationTimes.exactly(2));
```

_MockServer_ est également capable de s'appuyer sur des [templates de réponses](https://www.mock-server.com/mock_server/response_templates.html) (Mustache, Velocity...).

Comme mentionné plus tôt, _MockServer_ offre également des fonctionnalités de proxying HTTP/HTTPS, ce qui signifie qu'il peut agir comme un proxy pour les requêtes HTTP/HTTPS et les rediriger vers des serveurs réels tout en enregistrant les interactions.

Vous avez beaucoup de tests ? Vraiment beaucoup ? Et vous avez peur que _MockServer_ ne tiennent pas la charge ? Vous avez raison, c'est avant tout une application logicielle, mais rassurez-vous, il est clusterisable. Certes avec quelques contraintes :

* même port d'écoute pour chaque instance : utilisez alors la commande `replicas` de docker.
* file-system partagé entre toutes les instances : créez un volume.

Je vous conseille alors d'utiliser le framework _TestContainers-Spock_ pour configurer votre nombre d'instances.  
De quelles performances parle-t-on ici ? L'équipe de _MockServer_ donne tous les détails dans leur page ["Scalability & Latency"](https://www.mock-server.com/mock_server/performance.html).

Vous aimez voir ce qui est configuré dans _MockServer_ ? Il fournit également une WebUI. Pour y accéder rien de plus simple, ouvrez votre navigateur et allez sur `http(s)://<host>:<port>/mockserver/dashboard`. Vous pouvez également utiliser la méthode `mockServerClient.openUI()` pendant votre session de debug, il ouvrira alors une nouvelle fenêtre dans votre navigateur vers sa WebUI.

L'API externe à laquelle votre application fait appel, est sécurisée via TLS ? _MockServer_ sait gérer ça aussi.

Pour les aficionados de Kubernetes, _MockerServer_ a un chart Helm...

### Wiremock

_Wiremock_ est très similaire à _MockServer_ en termes de fonctionnalités. Il est également  intégrable dans _Testcontainers_.
Les étapes de son intégration dans les tests, en tant que conteneur Docker, sont quasiment similaires à _MockServer_ :

* Ajout de la dépendance `org.wiremock.integrations.testcontainers:wiremock-testcontainers-module`
* Annotation de la classe de test avec `@Testcontainers`
* Déclaration du conteneur "_wiremockServer_" dans la classe de test

  ```java
    @Testcontainers
    class WireMockIntegrationTest {
      
        @Container
        private static final WireMockContainer wiremockServer = new WireMockContainer("wiremock/wiremock")
            .withCopyFileToContainer(MountableFile.forClasspathResource("my_spec.json"), "/home/wiremock/mappings/my_spec.json");

        
        [...]
  ```

  où le fichier _"my_spec.json"_ contient par exemple :

  ```json
    {
      "request": {
        "method": "GET",
        "url": "/my_endpoint"
      },
      "response": {
        "status": 200,
        "body": "I am a mock!"
      }
    }
  ```

* Enfin, il suffit de requêter notre application pour vérifier qu'elle utilise bien le mock défini et que la réponse correspond à celle attendue.

Utiliser des fichiers de mapping "Requête/Réponse" est une des solutions pour configurer _WireMock_.  
L'utilisation de l'API REST d'administration exposée par ce dernier en est une autre et elle peut être utilisée pour une configuration dite "à la volée" pendant l'exécution des tests. Par exemple, à l'aide du framework _RestAssured_ :

```java
given().baseUri(wiremockServer.getHost()) // URL du mock
        .port(wiremockServer.getPort()) // port du mock
        .contentType(ContentType.JSON)
        .body("{\"request\": {\"method\": \"GET\", \"url\": \"/my_path\"}, \"response\": {\"status\": 200, \"body\": \"I am a mock!\"}}") // mapping requête/réponse
        .when()
        .post("/__admin/mappings"); // requête
```

Notre exemple d'utilisation de _WireMock_ est disponible [ici](https://github.com/Younup/MockinOnApiSDoor/blob/main/src/test/java/fr/younup/mock/WireMockIntegrationTest.java).

#### Encore faim ?

Vous souhaitez configurer très finemement _WireMock_ ? Par exemple, vous souhaitez qu'il réponde différemment s'il reçoit une requête POST dont le body contient un numéro de téléphone finissant par _400_ ? C'est possible ! Il suffit d'ajouter un matcher dans la requête configurée, tel que :

```json
"bodyPatterns": [{"matchesJsonPath": "$.phoneNumber", "matches": ".*400$"}]
```

Vous l'aurez compris, _WireMock_ offre une configuration flexible des réponses et offre ainsi un contrôle plus fin du comportement de l'API simulée.

_WireMock_ permet également de simuler des délais de réponse / de latence réseau, des pannes de service et du load balancing. Ces fonctionnalités permettent de simuler des scénarios plus complexes et plus réalistes.

Enfin, tout comme _MockServer_, _WireMock_ permet de vérifier les requêtes reçues et leurs nombres.

### OpenAPI Mock

_TestContainer_ fournit une librairie pour _MockServer_ et une librairie pour _Wiremock_. Mais il n'en fournit pas pour _OpenAPI-mock_. Peu importe, _TestContainer_ permet via ses _GenericContainer_ d'utiliser n'importe quelle image docker au sein de vos tests.

En termes de fonctionnalités, il est également bien moins fourni que les deux autres, il s'appuie essentiellement sur le contrat d'interface OpenAPI V3 et il dispose de plusieurs modes d'utilisation configurables via la variable d'environnement `OPENAPI_MOCK_USE_EXAMPLES` :

* `no` : les exemples de valeurs présents dans le contrat d'interface sont ignorés et les valeurs sont générées aléatoirement par le mock. Ces valeurs se basent également sur les restrictions présentes dans le contrat d'interface (_minLength_, _maxLength_, _nullable_, type : _string_, _integer_...).
* `if_present` : les exemples de valeurs sont utilisés s'ils sont présents dans le contrat d'interface, sinon les valeurs sont générées aléatoirement.
* `exclusively` : les exemples de valeurs présents dans le contrat d'interface sont exclusivement utilisés, ce qui implique de les avoir renseignés pour chaque champ.

_OpenAPI mock_ contient d'autres clés de configuration telles que par exemple le taux de probabilité de valeurs nulles générées.

```java
@Testcontainers
class OpenApiMockIntegrationTest {

  @Container    
  static final GenericContainer<?> openApiMock = new GenericContainer<>("muonsoft/openapi-mock:0.3.9")
          .withExposedPorts(8080)
          .withCopyFileToContainer(MountableFile.forHostPath("./src/test/resources/openAPI_CI.yml"), "/tmp/spec.yaml")
          .withEnv(new HashMap<>() {{
              put("OPENAPI_MOCK_SPECIFICATION_URL", "/tmp/spec.yaml");
              put("OPENAPI_MOCK_USE_EXAMPLES", "if_present");
          }});
```

Nous avons tenu à présenter cette solution, certes beaucoup moins fournie que les deux autres, pour sa simplicité d'utilisation. Elle peut facilement être intégrée pour mocker une application exposant une API REST très simple.

Notre exemple d'utilisation de _OpenAPI Mock_ est disponible [ici](https://github.com/Younup/MockinOnApiSDoor/blob/main/src/test/java/fr/younup/mock/OpenAPIMockIntegrationTest.java).

## A vous de choisir

Avoir des mocks dits "intelligents" est un vrai plus lorsque nous écrivons des tests d'intégration sur une application ayant des interactions avec des API REST externes.

_MockServer_ et _WireMock_ sont clairement deux solutions parmi les plus complètes du marché et elles sont utilisables et intégrables très facilement.

_OpenAPI Mock_ peut très vite tirer son épingle du jeu dès que nous souhaitons tester des API très simples ou tester les cas nominaux de chaque endpoint d'une API. Sa fonctionnalité de génération aléatoire de valeurs dans les réponses est un vrai plus.

**Documentations officielles :**

* [Documentation Officielle de MockServer](https://www.mock-server.com/#what-is-mockserver)
* [Documentation Officielle de Wiremock](https://wiremock.org/docs/)
* [Documentation Officielle de OpenAPI Mock](https://github.com/muonsoft/openapi-mock/blob/master/README.md)
