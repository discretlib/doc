+++
title = "Gestion des Pairs"
description = "Apprenez à lister les pairs connus et à gérer les droits de connection"
weight = 2
+++

*Discret* maintient une liste de pairs qui représente les utilisateurs que vous connaissez. Ces pairs proviennent de deux sources:
- les pairs que vous avez invités
- les pairs qui ont des *Rooms* communes avec vous. Par exemple si un pair vous a rajouté dans une *Room* avec plusieurs utilisateurs, *Discret* va récupérer la liste de ces pairs

Le modèle de données est le suivant:
```js
sys{
    Peer {
        pub_key: Base64 ,
        name: String default "anonymous",
    }
}
```
Il est a noter que **name** n'est pas forcement unique. C'est au développeur de l'application de fournir un modèle pour gérer une description plus complète des pairs. 

La **pub_key** est la clé de signature du pair qui peut être utilisée pour lui donner des droits dans une *Room*.

Par défaut, seul les pairs que vous avez invités ont le droit de se connecter à vos appareils. La liste des pairs authorisés est géré par l'entité:
```js
sys{
    AllowedPeer{
        peer: sys.Peer,
        meeting_token: Base64,
        last_connection: Integer default 0,
        status: String,
    }
}
```
Les pairs ayant le droit de se connecter auront leur **status** égal à **enabled**. Vous avez la possibilité d'interdir la connection à un pair en mettant la valeur **disabled**.
