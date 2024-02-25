---
layout: post
title: "Keycloak et la fédération d'identité à travers un SPI custom"
resume: "Le sujet de cet article est de présenter une des fonctionnalités fournie par Keycloak, le SPI ou Service Provider Interface."
image: "/assets/images/keycloak-spi.png"
readTime: "15"
excerpt: keycloak SPI java fédération
---

## Table des matières <!-- omit in toc -->

1. [Introduction](#introduction)
2. [Besoin : un SPI pour fédérer les identités à travers Keycloak](#besoin--un-spi-pour-fédérer-les-identités-à-travers-keycloak)
3. [Dépendances](#dépendances)
4. [Implémentation](#implémentation)
5. [Représentation du SPI dans Keycloak](#représentation-du-spi-dans-keycloak)

## Introduction

Keycloak est un logiciel open source de la famille des IAM (*Identity and Access Management*) permettant l'authentification unique avec gestion des identités et des accès, destiné aux applications et services modernes.

Keycloak assure la fédération des utilisateurs, l'authentification forte, la gestion des utilisateurs, l'autorisation fine...

Le sujet de cet article n'est pas de présenter Keycloak mais de présenter une des fonctionnalités qu'il fournit, le SPI ou *Service Provider Interface*.

## Besoin : un SPI pour fédérer les identités à travers Keycloak

Keycloak est conçu de manière à couvrir la majorité des cas d'utilisation d'une application.
Il est également personnalisable via un certain nombre d'interfaces permettant d'implémenter son propre écran de login (thème) ou encore son propre fournisseur d'identité (à intégrer au sein de Keycloak).

Imaginons l'application assez standard suivante : un backend, un frontend et une base de données.

![Step 1](/assets/2024/05/21/01-legacy-solution.png)

Ici le backend est responsable de gérer l'authentification des utilisateurs et le stockage de ces derniers dans sa propre base de données.

Imaginons ensuite que le client souhaite migrer sa partie frontend vers une solution open source, car elle est vieillissante et/ou elle est basée sur une technologie sous licence.

On lui propose alors un frontend développé par exemple en Angular, et on lui propose Keycloak en tant que gestionnaire d'identités.

La solution proposée au client est la suivante :

![Step 2](/assets/2024/05/21/02-new-solution.png)

Le client ne souhaite pas payer pour migrer tous les clients et leurs mots de passe dans Keycloak, mais il accepte que les utilisateurs soient obligés de changer leur mot de passe lors de leur première authentification au nouveau site web.

C'est à ce moment-là que nous pouvons proposer un SPI Keycloak custom pour répondre à son besoin : celui-ci sera en charge, lors de la première authentification d'un utilisateur d'appeler un service Web spécifique du backend afin de vérifier son identité, de récupérer un attribut (ici l'ID de l'utilisateur dans la base de données du backend).

Une fois l'identité de l'utilisateur vérifiée, il sera également en charge de lui demander de changer son mot de passe, et de stocker cet utilisateur ainsi que son attribut et son mot de passe, dans la base de données de Keycloak.

## Dépendances

Que vous utilisiez *Gradle* ou *Maven*, vous devez ajouter les dépendances suivantes avec les versions correspondantes à la version de Keycloak que vous utilisez :

* *org.keycloak:keycloak-core*
* *org.keycloak:keycloak-services*
* *org.keycloak:keycloak-server-spi:provided*
* *org.keycloak:keycloak-model-legacy*

## Implémentation

Pour implémenter ce SPI, commençons par implémenter les interfaces qu'il fournit : *UserStorageProviderFactory* et *UserStorageProvider*/*UserLookupProvider*/*CredentialInputValidator*.

Concentrons-nous d'abord sur l'implémentation de notre nouvelle factory qui devra permettre à un administrateur Keycloak de pouvoir configurer l'URL du Backend.

```java
public class CustomUserStorageProviderFactory implements UserStorageProviderFactory<MyCustomUserStorageProvider>{

    @Override
    public String getId(){
        return "CustomUserProvider";
    }

    /**
     * Création du Provider
     */
    @Override
    public CustomUserStorageProvider create(KeycloakSession session, ComponentModel model){
        boolean resetPwd = model.get(CONFIG_KEY_FORCE_PWD_RESET, true);
        return new CustomUserStorageProvider(session, model, new CustomUserService(session), resetPwd);
    }

    /**
     * Définition de l'affichage du SPI dans l'interface web Keycloak
     */
    @Override
    public List<ProviderConfigProperty> getConfigProperties(){
        return ProviderConfigurationBuilder.create()
            // Permet de configurer de l'URL du backend via l'interface Web Keycloak
            .property(
                "Backend", // nom
                "Backend URL", // label
                "The backend URL. Ex: http://localhost:8181", // help text
                ProviderConfigProperty.STRING_TYPE, // type
                "http://localhost:8181", // valeur par défaut
                null // options
            )
            // Permet de forcer ou non le changement de mot de passe de l'utilisateur
            .property(
                    "Force Reset Password at first authentication",
                    "Reset Password?",
                    "Force the user to reset his password at its first authentication",
                    ProviderConfigProperty.BOOLEAN_TYPE,
                    true,
                    null)
            .build;
    }
}
```

On peut voir qu'il n'y a rien de très compliqué, mais qu'il dépend de 2 autres classes, le provider `CustomUserStorageProvider` et le service `CustomUserStorageService`.

Le service doit contenir l'implémentation de votre appel au backend qui a pour but de récupérer des informations utilisateur : la validation du mot de passe saisi à la première authentification et la récupération d'attributs.

On ne verra donc pas son implémentation ici, car elle va différer en fonction du endpoint exposé par votre Backend.

Regardons plutôt comment implémenter notre *Provider*.

```java
public class CustomUserStorageProvider implements UserStorageProvider, UserLookupProvider, CredentialInputValidator {

    private final KeycloakSession session;
    private final ComponentModel model;
    private final CustomUserStorageService userService;
    private final boolean resetPwd;

    /**
     * Attribut de l'utilisateur à récupérer depuis le backend et à stocker (peut permettre de faire le lien entre Keycloak et le backend en cas de problème sur ce compte) 
     */
    private static final String USER_ATTRIBUTE_ID = "USER_ID";

    public CustomUserStorageProvider(KeycloakSession session, ComponentModel model, CustomUserStorageService userService, boolean resetPwd){
        this.session = session;
        this.model = model;
        this.userService = userService;
        this.resetPwd = resetPwd;
    }

    @Override
    public UserModel getUserById(RealmModel realm, String id){
        StorageId storageId = new StorageId(id);
        String username = storageId.getExternalId();
        return getUserByUsername(realm, username);
    }

    /**
     * L'appel à cette méthode permet de créer un utilisateur dans le stockage local de Keycloak, si celui-ci n'existe pas déjà
     */
    @Override
    public UserModel getUserByUsername(RealmModel realm, String username){
        return createAdapter(realm, username);
    }

    @Override
    public UserModel getUserByEmail(RealmModel realm, String email){
        return getUserByUsername(realm, username);
    }

    @Override
    public boolean supportsCredentialType(String credentialType){
        return PasswordCredentialType(credentialType);
    }

    @Override boolean isConfiguredFor(RealModel realm, UserModel user, String credentialType){
        return supportsCredentialType(credentialType);
    }

    @Override
    public boolean isValid(RealmModel realm, UserModel user, CredentialInput credentialInput){
        // On vérifie si l'utilisateur existe déjà dans le stockage local de Keycloak
        UserModel localUser = UserStoragePrivateUtil.userLocalStorage(session).getUserByUsername(realm, user.getUsername());
        
        // Si l'utilisateur existe mais qu'il ne contient pas l'attribut attendu, on doit appelé le backend
        if (locaUser != null && localUser.getAttributes().get(USER_ATTRIBUTE_ID) == null){
            // Mot de passe saisi par l'utilisateur qui sera envoyé au backend pour vérification (en clair ici pour l'exemple, mais à chiffrer pour plus de sécurité)
            String backendPassword = credentialInput.getChallengeResponse(); 
            
            // On appelle le backend pour récupérer l'utilisateur
            JsonNode retrievedUser = userService.getUserByUserName(user.getUsername(), backendPassword, model);

            // Si l'utilisateur a bien été récupéré, on lui ajoute l'attribut
            if (retrievedUser != null && !retrievedUser.isEmpty()){
                user.setSingleAttribute(USER_ATTRIBUTE_ID, retrievedUser.get("UserId").textValue()); // l'attribut dans la réponse est "UserId

                // Une fois l'attribut récupéré, on doit supprimer le lien entre notre UserStorageProvider et Keycloak, ce qui permettra à Keycloak de vérifier l'identité de l'utilisateur dans sa base de données lors de l'authentification au lieu de demander au backend.
                user.setFederationLink(null); 

                if (resetPwd) {
                    // Si requis, on force la réinitialisation du mot de passe lors de la première authentification de l'utilisateur
                    user.addRequiredAction(UserModel.RequiredAction.UPDATE_PASSWORD);
                }
                return true;
            }
            // Si l'utilisateur n'existe pas ou n'a pas pu être retrouvé dans le backend, on doit en supprimer toute trace dans Keycloak
            UserStoragePrivateUtil.userLocalStorage(session).removeuser(realm, user);
        }
        return false;
    }

    protected UserModel createAdapter(RealmModel realm, String username){
        // On récupère l'utilisateur depuis le stockage local de Keycloak
        UserModel localuser = UserStoragePrivateUtil.userLocalStorage(session).getUserByUsername(realm, username);

        // Si l'utilisateur n'existe pas dans le stockage local de Keycloak, on doit l'initialiser
        if (localUser == null){
            localUser = UserStoragePrivateUtil.userLocalStorage(session).addUser(realm, username);
            localUser.setEmail(username);
            localUser.setEmailVerified(true);
            localUser.setFederationLink(model.getId());
            localUser.setEnabled(true);
        }

        // On crée un nouvel Adapter (basé sur la classe AbstractUserAdapter)
        return new UserModelDelegate(localUser){
            @Override
            public String getUsername(){
                return username;
            }
            @Override
            public SubjectCredentialManager credentialManager(){
                return new UserCredentialManager(session, realm, this){};
            }
        };
    }
}
```

Vous l'aurez compris, la méthode la plus importante ici est `isValid()`.
C'est elle qui contient toute la logique de votre algorithme.
J'ai essayé de commenter au maximum l'implémentation pour que vous compreniez bien la logique.

Dernier point important, si vous voulez que votre SPI soit pris en compte par Keycloak, vous devez créer le fichier `org.keycloak.storage.UserStorageProviderFactory` dans `src/main/resources/META-INF/services` et il doit contenir le chemin et le nom de votre *ProviderFactory* custom `com.example.storage.CustomUserStorageProviderFactory`.

Il ne vous reste ensuite plus qu'à builder et à ajouter le jar généré par le build (package de votre SPI) dans le dossier `/opt/keycloak/provider` de votre installation de Keycloak (ou dans votre DockerFile si vous utilisez Keycloak en tant que conteneur Docker).

*Retrouvez cet exemple dans ce [repository GitHub](https://github.com/cedricSarre/keycloak-spi-user-storage).*

## Représentation du SPI dans Keycloak

Une fois votre instance de Keycloak déployée, connectez-vous à la console d'administration, sélectionnez votre royaume (*Realm*) et allez dans le menu "*User federation*" et vous constaterez que vous pouvez en ajouter un nouveau de votre type (dans l'image ci-dessous, de type "*CustomUserProvider*").

![Step 3](/assets/2024/05/21/03-user-federation.png)

Créez-en un nouveau ou sélectionnez-le s'il est déjà présent, et accédez aux détails.  
Comme le montre l'image ci-dessous, vous retrouverez vos paramètres avec les valeurs par défaut que vous avez configurées (et que vous pouvez modifier).

![Step 4](/assets/2024/05/21/04-user-federation-details.png)

Au final, c'est assez simple.  
Ici, nous stockons les utilisateurs dans Keycloak, mais vous pourriez également imaginer de continuer à utiliser votre backend comme stockage de vos vieux utilisateurs, le tout en utilisant Keycloak comme gestionnaire d'identité pour vos nouveaux utilisateurs.
