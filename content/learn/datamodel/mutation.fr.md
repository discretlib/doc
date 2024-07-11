+++
title = "Mutations et Suppression"
description = "Apprenez à inserer des données, les modifier et les supprimer"
weight = 2
+++

Les requêtes de type 'mutation' permettent d'insérer ou de modifier des données. 

Les exemples utiliseront le [schéma](@/learn/datamodel/schema.fr.md) suivant:
```js
{
    Person {
        name: String,
        surname: String nullable,
        parents:[Person]
    }
}
```

# Insérer
Lors de la creation d'un tuple, un identifiant est généré et stocké dans le champ **id**. Cet identifiant est retourné par la requête de *mutation*.

La syntaxe pour insérer un nouveau tuple est la suivante:
```js
mutate {
    Person {
        name : "Doe"
        surname: "John"
    }
}
```
Cette mutation insère une nouvelle personne nommée 'John Doe'.

Le résultat d'une requête de mutation retourne un object JSON comprenant les champs que vous avez insérés, ainsi que l'**id** nouvellement créé:
```js
{
    "Person":{
        "id":"aVBDUWlHpWz9bv_vU-Feow",
        "name":"Doe",
        "surname":"John"
    }
}
```


# Mutation multiples
Il est possible de faire plusieurs mutations en une seule requête:
```js
mutate {
    p1: Person {
        name : "Doe"
        surname: "John"
    }

    p2: Person {
        name : "Alice"
    }

    last: Person {
        name : "Bob"
    }
}
```
Les alias *p1, p2, last* peuvent être n'importe quelle chaine de caractères mais doivent être uniques pour la mutation, car chaque requête va retourner un champs JSON différent. Le résultat de la mutation sera l'object JSON suivant:

```js
{
    "last":{
        "id":"JoC5F5bxu-zQm5xer5o5-w",
        "name":"Bob"
    },
    "p1":{
        "id":"JoAWR7-zcUR5ri_yfZaqXQ",
        "name":"Doe",
        "surname":"John"
    },
    "p2":{
        "id":"JoCipHxxy7ha1xa2iZG5Ow",
        "name":"Alice"
        }
}

```
Vous noterez que pour des raisons techniques, l'ordre des champs JSON retournés n'est pas garanti d'être le même que pour la requête.


# Imbrication 
Il est possible d'insérer en une seule requête une entité et ses relations. Pour l'exemple de **Person**, la requête suivante va insérer 'John Doe' ainsi que ses parents.
```js
mutate {
    Person {
        name : "Doe"
        surname: "John"
        parents: [
            {
                name : "Alice"
            },{
                name : "Bob"
            }
        ]
    }
}
```

Vous pourrez noter que:
- la définition des deux parents est séparée par une virgule;
- il n'est pas nécessaire d'indiquer que Alice et Bob sont de type **Person** car cela et sous entendu par la définition du schéma.

Si les relations ont déja été insérées et que vous connaissez leur **id** il est possible de les utiliser lors de l'insertion d'un nouveau tuple
```js
mutate {
    Person {
        name : "Doe"
        surname: "John"
        parents: [
            {
                id : $mother_id //n'insère pas de nouveau tuple et 
                                //utilise le tuple référencé.
            },{
                id : "JoAWR7-zcUR5ri_yfZaqXQ" //n'insère pas de nouveau 
                                //tuple et utilise le tuple référencé.
            }
        ]
    }
}
```
Si les **id** fournits n'existent pas, une erreur est retournée.

# Mettre à jour 
Une requête de mise à jour est de type *mutation* qui contient l'identifiant du tuple à modifier.

```js
mutate {
    Person {
        id: $id
        surname: "Alice"
    }
}
```
Cette mutation va mettre à jour le champ surname du tuple ayant l'**id** indiqué. Si cet id n'existe pas, une erreur est retournée.

Il est possible d'ajouter des relations à un tuple existant:
```js
mutate {
    Person {
        id: $id
        parents: [
            {id:$mother_id}, 
            {
                name: "Bob"
            }
        ]
    }
}
```
Cette requête va ajouter au tuple reférencé par **$id**:
- une reférence au tuple défini par **$mother_id**
- un nouveau tuple **Person** ayant pour nom "Bob"

Il est aussi possible de supprimer toutes les relation d'un champ donné en lui passant la valeur **null**:
```js
mutate {
    Person {
        id: $id
        parents: null
    }
}
```
Cette requête supprime tous les parents du tuple référencé par **$id**


# Suppression de données
La suppression de données s'effectue en utilisant une requête de type **delete**.
il est possible de supprimer un tuple, ou bien une relation dans un champs du tuple.

La requête suivante supprimer un tuple ayant pour identitifiant **$id**:
```js 
delete {
    Person { $id }
}
```

La requête suivante supprime le parent ayant pour identifiant:$parent _id
```js 
delete {
    Person { 
        $id 
        parents[$parent _id]
    }
}
```


# Propagation de la *Room*
La plupart des données que vous allez insérer seront attaché à une [Room](@/learn/access_rights/room.fr.md) à l'aide du champ système **room_id**. Dans le cas d'une requête imbriquée, la **room_id** du tuple *parent* sera propagé aux tuples d'une relation si ceux ci n'ont pas de **room_id**. Cela permet de simplifier la syntaxe en cas de requête complexe.

Dans l'exemple suivant, "Alice" sera insérée dans la "room" ayant pour id **$room_id**.
```js
mutate {
    Person {
        room_id: $room_id
        name : "Doe"
        surname: "John"
        parents: [
            {
                name : "Alice"
            },{
                room_id: $second_room_id
                name : "Bob"
            }
        ]
    }
}
```
