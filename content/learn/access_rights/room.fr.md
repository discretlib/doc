+++
title = "Rooms"
description = "Apprenez à gérer les droits d'accès aux données"
weight = 1
+++

Les droits d'accès sont au coeur du mécanisme de synchronisation de *Discret*. Chaque tuple peut être assigné à une *Room* (*Salle* ou *Chambre* en anglais) qui définit un ensemble de droits: 
- qui peut modifier les données,
- qui peut y acccéder,
- quel type de donnée est valable dans cette *Room*,
- qui est administrateur de la *Room*.

Chaque tuple de données ne peut être que dans une seule *Room*, mais vous pouvez déplacer vos données d'une *Room* à l'autre. 

Lors de l'insertion ou modification d'un tuple dans cette *Room*, les droits d'accès seront utilisés pour vérifier que l'utilisateur a le droit de faire cette opération.

Lors de la connection avec d'autres *Pairs*, ces *Rooms* seront utilisées pour savoir quelle données doivent être synchronisées avec ces pairs. 

Enfin, lors de la synchronisation des données, chaque tuple sera vérifié pour garantir que les données reçues ont le droit d'exister dans cette *Room*. 


# Schéma

Le schéma d'une **Room** est le suivant:
```js
sys{
    Room {
        admin: [sys.UserAuth],
        authorisations:[sys.Authorisation]
    }
    
    Authorisation {
        name: String,
        rights:[sys.EntityRight] ,
        users:[sys.UserAuth],
        user_admin: [sys.UserAuth],
    }
    
    UserAuth{
        verif_key: Base64,
        enabled: Boolean default true,
    }
    
    EntityRight {
        entity: String,
        mutate_self: Boolean,
        mutate_all: Boolean,
    }
}
```

L'entité **Room** est l'entité parente qui contient le reste de la définition:
- **admin**: définit les administrateurs de la *Room* en question
- **authorisations**: définit les droit d'accès 

L'entité **Authorisation**  contient un groupe de droits d'accès à cette *Room*. Une *Room* peut contenir plusieurs authorisation:
- **name**: le nom de l'authorisation, par exemple: autheurs, lecteurs, etc.
- **rights**: les droits d'accès au niveau des entité: quelle entité est acceptée dans cette Room et quelles sont les authorisations d'écriture pour cette authorisation,
- **users**:  les utilisateurs authorisés,
- **user_admin**: qui peut gérer des utilsateurs de cette authorisation.

L'entité **UserAuth** définit un utilisateur:
- **verif_key**: la clé cryptographique de vérification de signature de l'utilisateur.
- **enabled**: permet d'activer ou de désactiver cet utilisateur
- **mdate**: le champ système **mdate** définit la date de début de validité de cette authorisation.

L'entité **EntityRight** définit les droits d'accèss pour une entité donnée:
- **entity**: le nom de l'entité incluant son espace de nom. Par exemple: **house.Person**. 
- **mutate_self**: indique si vous pouvez inserer de tuples de cette entité, et modifier les tuples que vous avez inséré ,
- **mutate_all**: indique si vous avez le droit de modifier des tuples créés par d'autre personne,
- **mdate**: le champ système **mdate** définit la date de début de validité de ce droit d'accès.

Tous les utilisateurs ayant accès à une *Room* ont des droit de lecture sur toutes les entités. Les **EntityRight** ne sont utilisés que pour définir les droits de modification.


Bien que cela ne soit pas recommandé, il est possible de définir un droit pour toutes les entité en insérant le *joker* **\*** dans le champ **entity**. Les droit d'accès définit avec le *joker* **\*** n'est pas prioritaire et ne sera utilisé que si l'entité n'a pas d'**EntityRight** propre. 

Pour garantir l'intégrité de la synchronisation au fil du temps, certaines contraintes sont appliquées:
- il n'est pas possible de supprimer, les entitités d'une room: par exemple, une fois qu'une **Authorisation** a été créée, elle ne peut plus être supprimée. 
- il n'est pas non plus possible de modifier les entités **UserAuth** et **EntityRight**. Par exemple, désactiver un **UserAuth** nécessite d'insérer un nouveau tuple avec la même **verif_key** et le champ **enabled** définit comme *false*. 

# Insérer des données dans une *Room*

Chaque tuple possède un champ système nommé **room_id** qui référence la *Room*.

Par exemple, en considérant que **$room_id** contient l'indentifiant d'une *Room* existante, la requête suivante va insérer un tuple **Person** dans la *Room* 
```js
mutate {
    Person{
        room_id: $room_id
        name: "Alice"
    }
}
```
Lors de l'insertion ce tuple sera signé avec votre clé de signature, et votre clé de vérification sera insérée dans le champs système **verifying_key**

Le système de vérification des droit d'accès va se baser sur cette **verifying_key** pour vérifier que vous avez le droit d'insérer une **Person** dans la *Room* référencée par **$room_id**


# Exemple: droits pour un *Blog* 

Le modèle de données utilisé est le suivant 
```js
blog {
    Article {
        title: String,
        content: String
    }

    Comment {
        article: blog.Article
        comment: String
    }
}
```

L'auteur du blog doit pouvoir:
- Insérer et modifier des **Articles**
- Insérer et modifier des **Comment**, y compris modifier des commentaires des lecteurs afn de pouvoir faire de la modération.

Le lecteur doit pouvoir:
- Lire les **Articles**
- Insérer des **Comment**

Dans la *Room* définissant les droits d'accès au blog, nous allons donc créer deux **Authorisations**
- authors: qui contiendra les droits des auteurs
- readers: qui contiendra les droits des lecteurs

La *Room* est créée avec la requête suivante la suivante:
```js, linenos
mutate {
    sys.Room{
        admin: [{
            verif_key:$admin_user
        }]
        authorisations:[{
            name:"authors"
            rights:[
                {
                    entity:"blog.Article"
                    mutate_self:true
                    mutate_all:true
                },
                {
                    entity:"blog.Comment"
                    mutate_self:true
                    mutate_all:true
                }
            ]
            users:[{
                verif_key:$author
            }]
        },{
            name:"readers"
            rights:[ {
                    entity:"blog.Comment"
                    mutate_self:true
                    mutate_all:false
            }]
            users:[{
                verif_key:$reader_1
            },{
                verif_key:$reader_1
            }]
        }]
    }
}
```

la ligne 



Vous noterez que nous n'avons pas défini de droits sur l'entité **Articles** pour les lecteurs, car toute personne ayant acces à la *Room* a un accès en lecture seule sur toutes les entités. 

# Exemple: un calendrier partagé 
Cet exemple est plus complexe que le précedent et montre une interaction entre plusieur **Rooms**.

Pour un calendrier partagé nous devons pouvoir séparer la visibilité entre la date d'un rendez-vous et son contenu. Par exemple:
- nous voudrions partager notre emploi du temps à tous les collaborateurs de notre entreprise mais sans partager les détails. Cela permet aux collaborateurs de connaître nos disponibilités.
- nous partageons les détails pour les quelques personnes de notre équipe , afin qu'ils sachent en plus ce que nous faisons.

Comme les utilisateurs d'une **Room** ont accès à toutes les donnée, nous avons besoin de deux *Rooms* différentes:
- une pour stocker les date de rendez-vous
- une pour stocker les détails des rendez-vous

Nous utiliserons ce modèle simplifié:
```js
cal {
    Calendar {
        name: String,
        appointments: [cal.Appointment],
    }

    Appointment {
        start: Integer,
        end: Integer,
        detail: cal.AppointmentDetail
    }

    AppointmentDetail {
        title: String,
    }
}
```


