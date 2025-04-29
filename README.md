# uOttawa - 2024-2025 - Projet

**Nom du projet** : Système de gestion de commandes de PC personnalisés

## Membres du projet

| Prénom |  Nom  |       Identifiant GitHub       |
|--------|-------|--------------------------------|
| Nicola | Baker | https://github.com/NicolaBaker |

## Introduction

Ce projet a pour but de créer une application qui facilite la gestion de commandes de PC personnalisés. L'application permet à plusieurs rôles d'interagir : **Administrateur**, **StoreKeeper**, **Assembler**, et **Requester**. Chaque rôle a des fonctionnalités pour garantir une gestion fluide des utilisateurs, du stock et des commandes, avec une authentification et une interface simple à utiliser.

##### Vidéo de l'application final : https://uottawa-my.sharepoint.com/personal/nbake013_uottawa_ca/_layouts/15/guestaccess.aspx?share=EY_8uhI2lIRIvExtw6GYLm0BJROYejZ2cpvIljB9534hRQ&nav=eyJyZWZlcnJhbEluZm8iOnsicmVmZXJyYWxBcHAiOiJPbmVEcml2ZUZvckJ1c2luZXNzIiwicmVmZXJyYWxBcHBQbGF0Zm9ybSI6IldlYiIsInJlZmVycmFsTW9kZSI6InZpZXciLCJyZWZlcnJhbFZpZXciOiJNeUZpbGVzTGlua0NvcHkifX0&e=7ls6Qo 
---

## Clarifications sur les exigences

### Exigences explicites reformulées
1. Chaque utilisateur doit se connecter avec un rôle spécifique.
2. Les administrateurs peuvent ajouter, modifier ou supprimer des utilisateurs.
3. Les gestionnaires de stock peuvent ajouter et gérer des composants matériels et logiciels.
4. Les assembleurs gèrent les commandes, les acceptent ou les rejettent.
5. Les demandeurs peuvent créer des commandes et suivre leur statut.
6. La base de données Firebase est utilisée pour l’authentification et la gestion des données.

### Exigences implicites proposées
1. Les informations doivent être mises à jour en temps réel avec Firebase.
2. Les champs de saisie doivent être vérifiés pour éviter les erreurs.
3. Les notifications visuelles (toasts) sont utilisées pour confirmer les actions.

### Hypothèses
1. Tous les utilisateurs sont autorisés par l’administrateur.
2. Une connexion Internet est nécessaire pour accéder aux fonctionnalités.
3. Les rôles sont bien définis et immuables une fois attribués.
---

## Modélisation

### Diagramme de classes
```plantuml
@startuml

    interface Serializable {
        
    }
    interface ComposantCallback {
        +onComposantAdded(Composant)
    }

    abstract class Utilisateur implements Serializable {
        + firstName: String
        + lastName: String
        + email: String
        + password: String
        + role: String
        + createdAt: Date
        + modifiedAt: Date

        + login(email: String, password: String)
        + logout()
    }

    abstract class Composant {
        + id: String
        + status: String
        + message: String
        + createdAt: Date
        + modifiedAt: Date
    }

    class AdminActivity extends Utilisateur {
        + usersList: ArrayList<Utilisateur>
        + emailField: EditText
        + passwordField: EditText
        + firstNameField: EditText
        + lastNameField: EditText

        + loadUsers()
        + saveUsers()
        + addUser(String, String, String, String, String)
        + deleteUser(String)
        + findUserByEmail(String)
        + resetFields()
    }

    class Materiel {
        + type: String
        + motherboard: String
        + memory: ArrayList<String>
        + storage: ArrayList<String>
        + display: ArrayList<String>
        + keyboardMouseSet: String

        + configureHardware()
    }

    class Logiciel {
        + webBrowser: String
        + officeSuite: String
        + devTools: ArrayList<String>

        + configureSoftware()
    }

    class StoreKeeperActivity implements ComposantCallback {
        + hardwareStock: ArrayList<Materiel>
        + softwareStock: ArrayList<Logiciel>
        + stock: Stock

        + viewStock()
        + addStock(Materiel, Logiciel)
        + deteleStock(Materiel, Logiciel)
        + updateStock()
    }

    class Commande extends Composant {
        + id: String
        + idUser: String

        + assignToRequester(String)
        + addCommande(Materiel, Logiciel)
        + deleteCommande()
        + getStatus(String)
    }

    class RequesterActivity extends Utilisateur {
        + commende: ArrayList<Commande>

        + addCommande(HardwareConfig, SoftwareConfig)
        + viewCommandes()
        + deleteCommande(String)
    }
    class Stock {
        + materiel: ArrayList<Materiel>
        + logiciel: ArrayList<Logiciel>

        + updateStock() : void
        + viewStock() : ArrayList<Composant>
    }
    class AdminFunctionActivity {
        + clearDatabase() : void
        + resetAllUsers() : void
        + deleteAllComposants() : void
        + deleteAllOrders() : void
    }

    class ComposantDetailActivity {
        + componentId: String
        + componentType: String
        + componentDetails: String

        + viewComposantDetails() : void
    }

    class ConsulterCommandesActivity {
        + commandesList: ArrayList<Commande>

        + viewAllCommandes() : ArrayList<Commande>
        + filterCommandes(status: String) : ArrayList<Commande>
    }

    class ModifierUtilisateur {
        + userId: String
        + user: Utilisateur
        + newRole: String
        + newEmail: String

        + modifyUserRole() : void
        + updateUserEmail() : void
        + updateUserDetails() : void
    }

    class AssemblerActivity extends Utilisateur {
        + allCommendes: ArrayList<Commande>

        + acceptCommendes(String)
        + rejectCommendes(String, String)
        + completeCommandes(String)
        + viewCommandes()
    }

    StoreKeeperActivity "1" -- "1" Stock

    RequesterActivity "1" -- "*" Commande
    AssemblerActivity "1" -- "*" Commande
    Stock "1" -- "*" Materiel
    Stock "1" -- "*" Logiciel

    AdminFunctionActivity "1" -- "1" AdminActivity
    AdminActivity "1" -- "*" ModifierUtilisateur
    ComposantDetailActivity "1" -- "*" Composant
    ConsulterCommandesActivity "1" -- "*" Commande

@enduml
```

### Diagrammes d'utilisation
```plantuml
@startuml
package "CustomDesktopService" #DDDDDD {
    actor Administrator
    actor StoreKeeper
    actor Assembler
    actor Requester

    "Register a requester" as (registerRequester)
    "Unregister a requester" as (unregisterRequester)
    "Add stock" as (addStock)
    "Remove stock" as (removeStock)
    "View stock" as (viewStock)
    "Create order" as (createOrder)
    "View orders" as (viewOrders)
    "Accept order" as (acceptOrder)
    "Complete order" as (completeOrder)
    "Delete order" as (deleteOrder)

    Administrator --> (registerRequester)
    Administrator --> (unregisterRequester)
    StoreKeeper --> (addStock)
    StoreKeeper --> (removeStock)
    StoreKeeper --> (viewStock)
    Requester --> (createOrder)
    Requester --> (viewOrders)
    Requester --> (deleteOrder)
    Assembler --> (acceptOrder)
    Assembler --> (completeOrder)
    Assembler --> (viewOrders)
}
@enduml

```
### Diagrammes d'activités

#### Accueil et authentification
```plantuml
@startuml
    title Authentification

    start
        while (Appui sur la touche de retour ?) is (Non) 
            :Affichage de la fenêtre d'accueil;
        
            :Saisie et validation de l'identifiant et du mot de passe;
        
            if (Authentification validée) then (Oui)
                If (L'utilisateur est un Administrator) then (Oui)
                    :Afficher la fenêtre d'un Administrator;
                elseif (L'utilisateur est un StoreKeeper) then (Oui)
                    :Afficher la fenêtre d'un StoreKeeper;
                elseif (L'utilisateur est un Assembler) then (Oui)
                    :Afficher la fenêtre d'un Assembler;
                elseif (L'utilisateur a le rôle Requester) then (Oui)
                    :Afficher la fenêtre d'un Requester;
                else
                :Erreur de conception: rôle inconnu;
                endif
            else (Non)
                :Afficher une erreur d'authentification;
            endif
        endwhile (Oui)

        :Libérer les ressources (base de données...);
    stop
@enduml
```
#### Gestion des utilisateurs
```plantuml
@startuml
    title Gestion des Utilisateurs

    start
        :Affichage de la fenêtre d'accueil;
        
        while (Tentatives d'authentification restantes > 0 ?) is (Oui)
            :Saisie et validation de l'identifiant et du mot de passe;
            
            if (Authentification validée) then (Oui)
                if (L'utilisateur est un Administrator) then (Oui)
                    :Afficher la fenêtre de l'Administrator;
                elseif (L'utilisateur est un StoreKeeper) then (Oui)
                    :Afficher la fenêtre du StoreKeeper;
                elseif (L'utilisateur est un Assembler) then (Oui)
                    :Afficher la fenêtre de l'Assembler;
                elseif (L'utilisateur a le rôle Requester) then (Oui)
                    :Afficher la fenêtre du Requester;
                else
                    :Erreur de conception: rôle inconnu;
                endif
                break
            else (Non)
                :Afficher une erreur d'authentification;
                :Décrementer les tentatives restantes;
            endif
        endwhile (Non)

        if (Tentatives d'authentification épuisées) then (Oui)
            :Afficher un message de blocage de compte;
        endif

        :Libérer les ressources (base de données);
    stop
@enduml
```


#### Gestion du stock
```plantuml
@startuml
    title Gestion du Stock

    start

    :Affichage de la fenêtre de gestion du stock;
    
    while (Action sélectionnée ?) is (Non)
        :Demander à l'utilisateur de choisir une action;
        
        if (Action = Ajouter un composant) then (Oui)
            :Saisie des détails du composant (ID, nom, quantité, description, etc.);
            :Vérifier si le composant existe déjà dans le stock;
            
            if (Composant existe) then (Oui)
                :Mettre à jour la quantité du composant existant;
            else (Non)
                :Ajouter un nouveau composant au stock;
            endif
            
        elseif (Action = Retirer un composant) then (Oui)
            :Demander l'ID du composant à retirer;
            :Vérifier si la quantité disponible est suffisante;
            
            if (Quantité suffisante) then (Oui)
                :Retirer la quantité spécifiée du stock;
            else (Non)
                :Afficher un message d'erreur : "Quantité insuffisante";
            endif
        
        elseif (Action = Mettre à jour un composant) then (Oui)
            :Demander l'ID du composant à mettre à jour;
            :Mettre à jour les informations (quantité, description, etc.);
        
        elseif (Action = Afficher le stock) then (Oui)
            :Afficher la liste des composants et leurs quantités;
        
        else
            :Afficher un message d'erreur : "Action inconnue";
        endif
        
    endwhile (Oui)
    
    :Libérer les ressources (base de données);
    
    stop
@enduml
```

#### Passage d'une commande
```plantuml
@startuml
    title Passage d'une commande

    start

    :Affichage de la fenêtre de sélection des composants;
    
    :Saisie des composants et des quantités souhaitées;
    
    :Vérification de la disponibilité des composants;
    
    if (Tous les composants sont disponibles) then (Oui)
        :Calculer le total de la commande;
        
        :Affichage du récapitulatif de la commande (produits, quantités);
        
        :Demander à l'utilisateur de confirmer la commande;
        
        if (Commande confirmée) then (Oui)
            :Enregistrer la commande dans le système;
            :Mettre à jour le stock en fonction des quantités commandées;
            :Envoyer la commande au service de traitement (préparation ou expédition);
            :Afficher un message de confirmation de commande passée;
        else (Non)
            :Annuler la commande;
            :Afficher un message d'annulation;
        endif
    else (Non)
        :Afficher un message d'erreur : "composants en rupture de stock";
        
    endif

    :Libérer les ressources (base de données);
    
    stop
@enduml
```

#### Traitement d'une commande
```plantuml
@startuml
    title Traitement d'une commande

    start

    :Affichage de la liste des commandes disponibles;
    :Sélection d'une commande à traiter;

    if (Commande en attente d'acceptation ?) then (Oui)
        :Afficher les détails de la commande;
        :Demander à l'assembleur d'accepter ou rejeter la commande;
        
        if (Commande acceptée ?) then (Oui)
            :Mettre à jour l'état à "en cours d'assemblage";
        else (Non)
            :Demander un commentaire;
            :Mettre à jour l'état à "rejetée";
        endif
    else (Non)
        if (Commande en cours d'assemblage ?) then (Oui)
            :Afficher les détails de la commande;
            :Confirmer la clôture de la commande;
            
            :Mettre à jour l'état à "livrée";
            :Consommer les composants du stock;
            :Mettre à jour la base de données du stock;
            :Afficher un message de confirmation;
        else
            :Afficher un message : "Commande déjà traitée ou état invalide";
        endif
    endif

    :Libérer les ressources (base de données);
    stop
@enduml


@enduml
```

### Diagrammes de séquences

#### Pour l'accueil et l'authentification
```plantuml
@startuml
    actor Inconnu

    control MainActivity
    control AdministratorActivity
    control StoreKeeperActivity
    control AssemblerActivity
    control RequesterActivity

    Inconnu --> MainActivity : Demande d'authentification\n(avec identifiant et mot de passe)

    MainActivity <--> Database : Rechercher un utilisateur\navec un identifiant et un mot de passe

    alt L'utilisateur existe
        MainActivity <--> Database : Obtenir des informations sur l'utilisateur\n(dont son rôle) 
        
        alt Le rôle de l'utilisateur est Administror
            MainActivity --> AdministratorActivity
        alt Le rôle de l'utilisateur est StoreKeeper
            MainActivity --> StoreKeeperActivity
        alt Le rôle de l'utilisateur est Assembler
            MainActivity --> AssemblerActivity
        alt Le rôle de l'utilisateur est Requester
            MainActivity --> RequesterActivity
        else Rôle inconnu 
            MainActivity --> Inconnu : Afficher une erreur de conception
        end
    else Sinon
        MainActivity --> Inconnu : Afficher une erreur d'authentification
    end

    database Database
@enduml
```
#### Pour le rôle Administrator
```plantuml

@startuml
    actor Administrator

    control AdministratorActivity
    
    database Database

    Administrator --> AdministratorActivity : Créer un utilisateur

    AdministratorActivity <--> Database : Obtenir la liste des utilisateurs

    alt L'utilisateur existe déjà
        AdministratorActivity --> Database : Ajouter une ligne à la table Users
    else Sinon
        AdministratorActivity --> Administrator: Afficher une erreur
    end
@enduml
```
#### Pour le rôle StoreKeeper
```plantuml
@startuml
    actor Inconnu

    control MainActivity
    control StoreKeeperActivity
    control StockAdapter
    control AddComposantDialog
    control ComposantDetailActivity

    Inconnu --> MainActivity : Demande d'authentification\n(avec identifiant et mot de passe)

    MainActivity <--> Database : Rechercher un utilisateur\navec un identifiant et un mot de passe

    alt L'utilisateur existe
        MainActivity <--> Database : Obtenir des informations sur l'utilisateur\n(dont son rôle)
        
        alt Le rôle de l'utilisateur est StoreKeeper
            MainActivity --> StoreKeeperActivity : Accès à la gestion du stock
        else Rôle inconnu
            MainActivity --> Inconnu : Afficher une erreur de conception
        end
    else Sinon
        MainActivity --> Inconnu : Afficher une erreur d'authentification
    end

    StoreKeeperActivity --> AddComposantDialog : Ajouter un composant
    AddComposantDialog <--> Database : Sauvegarder le nouveau composant

    StoreKeeperActivity --> StockAdapter : Consulter la liste des composants
    StockAdapter <--> Database : Récupérer les composants du stock

    StoreKeeperActivity --> ComposantDetailActivity : Consulter les détails d'un composant
    ComposantDetailActivity <--> Database : Mettre à jour ou supprimer un composant

    alt Composant endommagé ou volé
        ComposantDetailActivity --> Database : Supprimer le composant
    end

    database Database
@enduml
```
#### Pour le rôle Assembler
```plantuml
@startuml
    actor Inconnu

    control MainActivity
    control AssemblerActivity
    control CommandesAdapter
    control CommandeDetailActivity
    control StockManager

    Inconnu --> MainActivity : Demande d'authentification\n(avec identifiant et mot de passe)

    MainActivity <--> Database : Rechercher un utilisateur\navec un identifiant et un mot de passe

    alt L'utilisateur existe
        MainActivity <--> Database : Obtenir des informations sur l'utilisateur\n(dont son rôle)
        
        alt Le rôle de l'utilisateur est Assembler
            MainActivity --> AssemblerActivity : Accès à la gestion des commandes
        else Rôle inconnu
            MainActivity --> Inconnu : Afficher une erreur de conception
        end
    else Sinon
        MainActivity --> Inconnu : Afficher une erreur d'authentification
    end

    AssemblerActivity --> CommandesAdapter : Visualiser toutes les commandes
    CommandesAdapter <--> Database : Récupérer les commandes

    AssemblerActivity --> CommandeDetailActivity : Consulter les détails d'une commande

    alt Commande « en attente d’acceptation »
        CommandeDetailActivity --> Database : Accepter ou rejeter la commande
        alt Commande acceptée
            CommandeDetailActivity --> Database : Passer à l’état « en cours d’assemblage »
        else Commande rejetée
            CommandeDetailActivity --> Database : Ajouter un commentaire et rejeter
        end
    end

    alt Commande « acceptée, en cours d’assemblage »
        CommandeDetailActivity --> StockManager : Consommer les composants nécessaires
        StockManager <--> Database : Mettre à jour le stock
        CommandeDetailActivity --> Database : Passer à l’état « livrée »
    end

    database Database
@enduml
```
#### Pour les commandes
```plantuml
@startuml
    actor Inconnu

    control MainActivity
    control RequesterActivity
    control ConsulterCommandesActivity
    control CommandesAdapter

    Inconnu --> MainActivity : Demande d'authentification\n(avec identifiant et mot de passe)

    MainActivity <--> Database : Rechercher un utilisateur\navec un identifiant et un mot de passe

    alt L'utilisateur existe
        MainActivity <--> Database : Obtenir des informations sur l'utilisateur\n(dont son rôle)
        
        alt Le rôle de l'utilisateur est Requester
            MainActivity --> RequesterActivity : Accès à la gestion des commandes
        else Rôle inconnu
            MainActivity --> Inconnu : Afficher une erreur de conception
        end
    else Sinon
        MainActivity --> Inconnu : Afficher une erreur d'authentification
    end

    RequesterActivity --> CommandesAdapter : Créer une commande de PC sur mesure
    CommandesAdapter <--> Database : Sauvegarder la commande

    RequesterActivity --> ConsulterCommandesActivity : Consulter les commandes passées
    ConsulterCommandesActivity <--> Database : Récupérer les commandes de l'utilisateur
    ConsulterCommandesActivity --> CommandesAdapter : Afficher la liste des commandes

    alt Commande en attente, rejetée ou livrée
        ConsulterCommandesActivity --> Database : Supprimer la commande
    else Commande en cours d'assemblage
        ConsulterCommandesActivity --> RequesterActivity : Affichage non modifiable
    end

    database Database
@enduml

```
### Diagrammes de état

#### Pour les commandes
```plantuml
@startuml
    state "Nouvelle Commande" as NouvelleCommande {
        
    }

    state "En cours de traitement" as EnCoursTraitement {
        EnCoursTraitement : Commande soumise
        NouvelleCommande --> EnCoursTraitement : soumettreCommande()
    }

    state "Acceptée" as Acceptee {
        Acceptee : Commande validée
        EnCoursTraitement --> Acceptee : validerCommande()
    }

    state "Rejetée" as Rejetee {
        Rejetee : Problème détecté (manque de stock ou autre)
        EnCoursTraitement --> Rejetee : rejeterCommande()
    }

    state "Livrée" as Livree {
        Livree : Commande livrée à l'utilisateur
        Acceptee --> Livree : livrerCommande()
    }

    state "Restaurée" as Restauree {
        Restauree : Commande soumise à nouveau
        Rejetee --> Restauree : restaurerCommande()
        Restauree --> EnCoursTraitement : soumettreCommande()
    }

    state "Archivée" as Archivee {
        Archivee : Commande terminée, aucune modification permise
        Livree --> Archivee : archiverCommande()
    }

    [*] --> NouvelleCommande
    Rejetee --> [*]
    Archivee --> [*]

@enduml

```
## Eléments de conception

Les éléments de conception incluent toutes les classes et fonctions nécessaires au fonctionnement de l'application. Les classes principales comme Utilisateur et Composant et Commande servent de base, tandis que des classes spécifiques comme AdminActivity, StoreKeeperActivity, RequesterActivity et AssemblerActivity ajoutent des fonctions adaptées à chaque rôle. Des activités comme ComposantDetailActivity et ConsulterCommandesActivity permettent de consulter ou gérer les détails des composants et des commandes. Chaque élément joue un rôle pour assurer que l'application soit fluide et réponde aux besoins des utilisateurs.

## Eléments de tests unitaires

Tous les tests unitaires ont passé avec succès. Cela signifie que toutes les fonctionnalités testées, comme les getters, setters et la mise à jour des données, fonctionnent correctement. Aucun problème n’a été trouvé lors de l'exécution des tests, et toutes les valeurs sont bien mises à jour comme prévu. Les tests ont également vérifié les valeurs par défaut pour les objets créés avec le constructeur sans paramètres. En résumé, tout est opérationnel et stable, sans erreur.
Outils Utilisés : JUnit
Comment Lancer les Tests :  test → "Run" pour exécuter les tests

### Espresso : 
Outil pour tester l'interface utilisateur Android de manière automatisée.
Tous les tests, y compris ceux d'interface utilisateur avec Espresso, ont été tester.

## Comment installer et utiliser la solution

Pour utiliser la solution téléchargez le projet, ouvrez-le dans Android Studio, configurez Firebase, puis lancez l’application sur un appareil ou un émulateur. Connectez-vous pour accéder aux fonctionnalités comme la gestion des stocks et des commandes.

## Eléments de démonstration

### Scénario ("storyboard") suggéré

Un demandeur peut créer une nouvelle commande en sélectionnant des composants matériels et logiciels, puis soumettre la commande. L'assembleur voit les commandes en attente, les accepte et met à jour leur statut une fois terminées. L'administrateur peut gérer les utilisateurs, modifier leurs informations ou supprimer des comptes si nécessaire. Le responsable du stock vérifie les quantités disponibles et ajoute ou retire des composants selon les besoins.

### Valeurs de test

#### Utilisateurs

| Rôle           | Identifiant de connexion | Mot de passe |
|----------------|--------------------------|--------------|
| Administrateur |      admin@baker.ca      |    Anicola   |
| StoreKeeper    |   storekeeper@baker.ca   |    Snicola   |
| Assembler      |    assembler@baker.ca    |    Asnicola  |
| Requester      |    r@baker.ca            |    nicola    |

#### Fichier de données exemple

Localisation : data/stock.json
Format : JSON contenant des informations sur le stock, les utilisateurs et les commandes.

## Limites et problèmes connus

L'interface pourrait être améliorée avec plus d'animations et un design plus moderne.

## Information destinées aux correcteurs

#### APK : PCSurMesure.apk
#### Identifiants de connexion : Voir la section des valeurs de test.
---


# Checklist pour le Projet livrable 3

## Notation de base
- [X] Chaque membre a effectué au moins 1 commit.


---

## Modèle Conceptuel UML
- [X] Le modèle conceptuel est fourni (Word, PDF, Markdown, Opuml, Visual Paradigm, etc.).
- [X] Il inclut au minimum un diagramme de classe UML :
  - [X] Comporte les principaux concepts du projet.
  - [X] Comporte au moins 10 attributs et 10 méthodes.
  - [X] Respecte le formalisme UML.
 

### Bonus :
- [X] +5 si le modèle comporte un diagramme d’activité.
- [ ] +5 si le modèle comporte un diagramme de séquence.
- [ ] +5 si le modèle comporte un diagramme d’états.

---

## Projet Android Studio
- [X] Le projet Android Studio est soumis avec l'APK de démonstration, le fichier README et le tag correspondant au livrable.

---

## Fonctionnalités de Gestion de Commandes
- [X] Les fonctions de gestion de commande (ajout, suppression, visualisation) sont en place et fonctionnelles.

---

## Tests Unitaires Automatisés (API)
- [X] Des tests unitaires automatisés (au moins 10) de niveau API sont en place pour valider la gestion des commandes.


---

## Validation des Champs
- [X] Tous les champs sont validés (valeurs saisies vérifiées, messages d'erreur appropriés en cas de saisie incorrecte).


---

## Bonus CircleCI
- [ ] La construction de l’application et les tests automatiques sont automatisés au moyen de CircleCI.


---

## Tests d’Interface Utilisateur (Espresso)
- [X] Des tests d’interface utilisateur basés sur Espresso sont en place.

---

## Organisation du Projet
- [X] Le projet est "propre" (bien organisé, facilement reconstructible, sans fichiers superflus et sans artefacts de reconstruction).

---

## Fichier README
- [X] Le projet contient un fichier README "riche" (avec beaucoup d'informations utiles) et bien présenté.

# Checklist pour le Projet Livrable 4

## Notation de base  
- [X] Chaque membre a effectué au moins 1 commit.  

---

## Modèle Conceptuel UML  
- [X] Le modèle conceptuel est fourni (Word, PDF, Markdown, Opuml, Visual Paradigm, etc.).  
- [X] Il inclut au minimum un diagramme de classe UML :  
    - [X] Comporte les principaux concepts du projet.  
    - [X] Comporte au moins 10 attributs et 10 méthodes.  
    - [X] Respecte le formalisme UML.
    - [X] le modèle comporte un diagramme d’activité.
    - [X] le modèle comporte un diagramme de séquence.  
    - [X] le modèle comporte un diagramme d’états.  

---

## Projet Android Studio  
- [X] Le projet Android Studio est soumis avec l'APK de démonstration, le fichier README et le tag correspondant au livrable.  

---

## Fonctionnalités de Gestion d'Assemblage
- [X] Les fonctions d’assemblage sont en place et fonctionnelles :  
  - [X] Consultation des commandes en attente.  
  - [X] Validation des commandes selon le stock disponible.  
  - [X] Mise en attente des commandes avec stock insuffisant.  
  - [X] Rejet des commandes avec raison justifiée.
  - [X] Approbation des commandes et simulation de livraison.  

---

## Tests Unitaires Automatisés (API)  
- [X] Des tests unitaires automatisés (au moins 10) de niveau API sont en place pour valider la gestion des commandes.

---

## Validation des Champs
- [X] Tous les champs sont validés (valeurs saisies vérifiées, messages d'erreur appropriés en cas de saisie incorrecte).

---

## Rapport Final
- [X] Le rapport final est complet et contient :
  - [X] Une référence au dépôt GitHub.
  - [X] Une description des choix de conception.
  - [X] Une section sur les exigences supplémentaires (proposées, traitées ou écartées).  
  - [X] Un retour d’expérience détaillé.
  - [X] Un tableau de synthèse des contributions.
  - [X] Une estimation du temps global passé sur chaque livrable.  

---

## Organisation du Projet
- [] Le projet est "propre" (bien organisé, facilement reconstructible, sans fichiers superflus et sans artefacts de reconstruction).  

---

## Fichier README  
- [X] Le projet contient un fichier README "riche" (avec beaucoup d'informations utiles) et bien présenté.  
