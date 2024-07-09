+++
title = "Requêtes"
description = "Decouvrez le moteur de requêtes et toute ses options"
weight = 3
+++

*Discret* fournit un moteur de requêtes assez complet et inspiré de GraphQL. 

La plupart des exemples utiliseront le [schéma](@/learn/datamodel/schema.fr.md) suivant:
```js
{
    Person {
        name: String,
        surname: String nullable,
        parents:[Person],
        pet: Pet,
    }

    Pet{
        name: String
    }
}
```

Avec le tuple suivant:
```js
 mutate {
    Person {
        name : "Doe"
        surname: "John"
        parents:  [
            {
                name : "Doe"
                surname: "Alice"
            } 
            ,{
                name: "Doe" 
                pet:{ name:"Kiki" }
            }
        ]
        pet: {name:"Truffle"}
        
    }
}
```


# Syntaxe

Commençons par faire une requête simple qui va recupérer le nom et prénom de toutes les personnes présentes dans la base de données:
```js
query {
    result: Person {
        name
        surname
    }
}
```
Cette requête retournera l'object JSON suivant:
```json
{
    "result":[
        {
            "name":"Doe",
            "surname":"John"
        },
        {
            "name":"Doe",
            "surname":"Alice"
        },
        {   "name":"Doe",
            "surname":null
        }
    ]
}
```

Il faut noter que:
- **result** est un alias, il peut être nommé différement
- les champs demandés *ne sont pas* délimités par une virgule,
- le résultat de la requete est contenu dans un tableau JSON, cela est vrai même si la requête ne renvoie qu'un seul tuple,
- si la base de donnée ne contient aucune donnée, la requête retourne un tableau vide.

# Requête Multiples
Il est possible de faire plusieurs requêtes en une seule fois.

La requête suivante va recupérér la liste des personnes et la liste des familiers:
```js
query {
    persons: Person {
        name
        surname
    }

    pets: Pet{
        name
    }
}
```

et retourne l'objet JSON suivant:
```json
{
    "persons":[
        {"name":"Doe","surname":"John"},
        {"name":"Coop","surname":"Alice"},
        {"name":"Doe","surname":null}
    ],

    "pets":[
        {"name":"Truffle"},
        {"name":"Kiki"}
    ]
}
```

Les alias **persons** et **pets** doivent être uniques au sein de la requête. 

# Filtrer
Les exemples précedents récupèrent la totalité des tuples correspondant a une entité. Dans la realité, les données doivent être filtrées.

Les filtres supportent les opérateurs suivants:
- **=**     égal
- **!=**    différent de
- **\>**     supérieur à
- **\>=**    supérieur ou égal à
- **<**     inférieur à
- **<=**    inférieur ou égal à


La syntaxe est la suivante:
```js
query {
    Person (
        name="Doe"
        ) {
        name
        surname
    }
}
```
Ici l'entité **Person** est filtrée en indiquant que l'on ne veut récupérer que les tuples dont le champ "nom" correspond à "Doe". Cela produit le résultat suivant: 
```json
{
    "Person":[
        {"name":"Doe","surname":"John"},
        {"name":"Doe","surname":null}
    ]
}
```

Il est possible de définir plusieurs filtre en les séparant par des virgules. 
```js
query {
    Person (
        surname="John",
        name="Doe"
        ) {
        name
        surname
    }
}
```
```json
{
    "Person":[{"name":"Doe","surname":"John"}]
}
```

Il est aussi possible de filter avec la valueur **null**
```js
query {
    Person (
        surname != null,
        name="Doe"
        ) {
        name
        surname
    }
}
```
```json
{
    "Person":[{"name":"Doe","surname":"John"}]
}
```




# Requêtes imbriquées

# Relations "Nullables" 


# Ordonancer

# Limiter le nombre de résultats 

# Pagination

# Aggrégation

# Recherche plein texte

# Sélecteurs Json 


