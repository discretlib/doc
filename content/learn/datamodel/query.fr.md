+++
title = "Requêtes"
description = "Découvrez le moteur de requêtes et toute ses options"
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

Avec les données suivantes:
```js
 mutate {
    Person {
        name : "Doe"
        surname: "John"
        parents:  [
            {
                name : "Coop"
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

Commençons par faire une requête simple qui va récupérer le nom et prénom de toutes les personnes présentes dans la base de données:
```js
query {
    result: Person {
        name
        surname
    }
}
```
Il faut noter que:
- **result** est un alias, il peut être nommé différemment,
- les champs demandés *ne sont pas* délimités par une virgule.

Cette requête retournera l'objet JSON suivant:
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
- le résultat de la requête est contenu dans un tableau JSON, cela est vrai même si la requête ne renvoie qu'un seul tuple,
- si la base de donnée ne contient pas les données demandées, la requête retourne un tableau vide.

# Requête Multiples
Il est possible de faire plusieurs requêtes en une seule fois.

La requête suivante va récupérer la liste des personnes et la liste des familiers:
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
Les exemples précédents récupèrent la totalité des tuples correspondant à une entité. Dans la réalité, les données doivent être filtrées.

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
Ici, l'entité **Person** est filtrée en indiquant que l'on ne veut récupérer que les tuples dont le champ "nom" correspond à "Doe". Cela produit le résultat suivant: 
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

Il est aussi possible de filtrer avec la valeur **null**
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


# Recherche plein texte
Par défaut, toute les données texte sont indexées pour pouvoir effectuer des recherches plein texte en utilisant la clause **search**

L'index utilisé est un index *trigram*, donc seule les chaines de plus de 3 caractères peuvent être recherchées.

L'indexation peut être [désactivée pour une entité](@/learn/datamodel/schema.fr.md#desactiver-l-indexation-plein-texte).

```js
query {
    Person(
        search("lic") //Alice contient lic
    ) {
        name
        surname
    }
}
```
Cette recherche retourne uniquement le tuple contenant "Alice".
```json
{
"Person":[{"name":"Coop","surname":"Alice"}]
}
```

# Requêtes imbriquées
Les requêtes ne sont pas limitées à la récupération d'un seul type d'entité. Au sein d'une même requête il est possible de récupérer une entité et ses relations.

la requête suivante récupère les personnes et leur familier:
```js
query {
    result: Person {
        name
        surname
        pet {
            name
        }
    }
}
```
Elle produit le résultat suivant: 
```json
{
    "result":[
        {   
            "name":"Doe",
            "surname":"John",
            "pet":{
                "name":"Truffle"
            }
        },
        {   
            "name":"Doe",
            "surname":null,
            "pet":{
                "name":"Kiki"
            }
        }
    ]
}
```

La requête renvoie uniquement les **Person** qui possèdent un **Pet**. Cela permet de garantir que votre requête renverra toujours des données non nulles, ce qui facilite le traitement.

Des filtres peuvent êtres appliqués aux sous-entités, comme pour l'entité principale: 
```js
query {
    Person {
        name
        surname
        parents (surname = null) {
            name
            pet (name = "Kiki"){
                name
            }
        }
        pets: {
            name
        }
    }
}
```

Cette requête produit le résultat suivant:
```json
{
    "Person":[
        {
            "name":"Doe",
            "surname":"John",
            "parents":[
                {
                    "name":"Doe",
                    "pet":{
                        "name":"Kiki"
                    }
                }
            ],
            "pet":{
                "name":"Truffle"
            }
        }
    ]
}

```

# Relations "Nullables" 
Pour certaines requêtes vous pouvez vouloir récupérer les entités ayant des relations vides. Pour ce faire if faut définir le champs de relation comme **nullable**:


```js
query {
    Person(
        nullable (pet)
    ) {
        name
        surname
        pet {
            name
        }
    }
}
```
Ici le champ **pet** est définit comme **nullable** et la requête retournera aussi les **Person** qui n'ont pas de **Pet**.

```json
{
    "Person":[
        {   "name":"Doe","surname":"John",
            "pet":{"name":"Truffle"}
        },
        {   "name":"Coop","surname":"Alice",
            "pet":null
        },
        {   "name":"Doe","surname":null,
            "pet":{"name":"Kiki"}
        }
    ]
}
```

Une fois un champ définit comme **nullable**, il est possible d'ajouter un filtre pour récupérer uniquement les tuples ayant une relation nulle: 
```js
query {
    Person(
        nullable (pet),
        pet = null
    ) {
        name
        surname
        pet {
            name
        }
    }
}
```
```json
{
    "Person":[
        {
            "name":"Coop",
            "surname":"Alice",
            "pet":null
        }
    ]
}
```

# Trier les résultats
Lorsqu'une requête retourne plusieurs tuples, l'ordre des tuples n'est pas garanti. Le tri de résultats se fait avec la clause **order_by**.

Le tri peut se faire sur plusieurs champs. Chaque champ doit avoir une indication de direction de tri:
- **asc** du plus petit au plus grand (ascendant)
- **desc** du plus grand au plus petit (descendant)

```js
query {
    Person(
        order_by(name desc, surname asc)
    ) {
        name
        surname
    }
}
```
```json
{
    "Person":[
        {"name":"Doe","surname":null},
        {"name":"Doe","surname":"John"},
        {"name":"Coop","surname":"Alice"}
    ]
}
```



Comme pour les filtres, **order_by** peut être définit pour les sous-entités.

# Limiter le nombre de résultats 
Si le nombre de tuples est très large, il peut être utile de limiter le nombre de tuples retournés en utilisant la clause **first**

```js
query {
    Person(
        order_by(name desc, surname asc),
        first 2
    ) {
        name
        surname
    }
}
```

Ici, seule les deux premiers tuples sont retournés
```json
{
    "Person":[
        {"name":"Doe","surname":null},
        {"name":"Doe","surname":"John"}
    ]
}
```

Il est aussi possible de *passer* un certain nombre de tuple en utilisant la clause **skip**

```js
query {
    Person(
        order_by(name desc, surname asc),
        skip 1, 
        first 2
    ) {
        name
        surname
    }
}
```

Ici, le premier tuple est ignoré, et les deux suivants sont retournés:
```json
{
    "Person":[
        {"name":"Doe","surname":"John"},
        {"name":"Coop","surname":"Alice"}
    ]
}
```


# Pagination
Il peut être tentant d'utiliser les clauses **skip** et **first** pour paginer de grands tableaux de résultats, mais cette méthode peut se reveler très lente car la clause **skip** peut avoir à itérer sur un très grand nombre de tuples. 

*Discret* fournit une alternative plus efficace avec les clauses **before** et **after**. Ces clauses fonctionnent en association avec la clause **order_by** et permettent de demander de filtrer les tuples retournés. 

La syntaxe est la suivante:

```js
query {
    Person(
        order_by(mdate desc, id desc),
        after ($date, $id) 
    ) {
        name
        surname
    }
}
```
Dans cette requête, les valeurs retournée seront celles dont la **mdate** est supérieure au paramètre **\$date** et dont l'**id** est supérieur à **\$id** si les dates sont égales.

Cette méthode de pagination est plus souple et bien plus rapide que l'utilisation de **skip** et **first**.

# Sélecteurs Json 
Les champs de types JSON peuvent êtres accédés à l'aide du sélecteur json **->$.**.


Considérons l'entité suivante contenant un champ JSON:
```js
{
    Article {
        content: json
    }
}
```

et insérons le tuple suivant:
```js
mutate {
    Article {
        content : "{
            \"title\": \"lorem ipsum\",
            \"content\": \"Rerum earum culpa quis. Corporis unde aliquid.\"
        }"
    }
}
```

La requête suivante retournera le titre de l'article:
```js
query sample{
    Article{
        title: content->$.title
    }
}
```
```json
{
    "Article":[
        {"title":"lorem ipsum"}
    ]
}
```




# Agrégation

*Discret* fournit les fonctions d'agrégation de données suivantes.**(field)** représente le nom du champ à agréger.
- **avg(field)**
- **count()**
- **max(field)**
- **min(field)**
- **sum(field)**

Un champ d'agrégation se présente sous la forme **alias: fonction()**. Si une entité définit un champ d'agrégation, tous les autres champs doivent aussi être des champs agrégés.

La requête suivante compte le nombre de parents pour une personne: 
```js
    query {
        Person (
            nullable(parents)
        )
        {
            name
            parent_count: parents {
                total: count()
            }
            parents {
                name
            }
        }
    }
```
et donne le résultat suivant:
```json
{
    "Person":[
        {   
            "name":"Doe",
            "parent_count":[{"total":2}],
            "parents":[{"name":"Coop"},{"name":"Doe"}]
        },
        {   
            "name":"Coop",
            "parent_count":[{"total":0}],
            "parents":[]
        },
        {   
            "name":"Doe",
            "parent_count":[{"total":0}],
            "parents":[]
        }
    ]
}
```

On note que le champ **parents** est utilisé deux fois:
- une fois avec l'alias **parent_count** pour effectuer la fonction d'aggrégation,
- une fois pour récupérer le nom des parents. 

Cela est nécessaire car on ne peut pas mélanger champs aggrégés et champs normaux. La requête suivante mélangeant **name** et **count()** n'aurait pas de sens:
```js
    query {
        Person (
            nullable(parents)
        )
        {
            name
            //cette requête n'a pas de sens et génère une erreur
            parents {
                total: count()
                name
            }
        }
    }
```




