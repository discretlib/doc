+++
title = "Schémas et Entités"
description = "Apprenez à créer et modifier un modèle de données"
weight = 1
+++

Discret utilise un language de schémas pour décrire les données que vous pouvez insérer.


# Syntaxe 

Commençons par un exemple simple:
```js
{
    Person {
        name: String,
        nickname: String nullable,
        parents:[Person]
    }
}
```
Nous avons défini une entité nommée **Person** qui contient trois champs:
- **name** qui doit contenir un type **String**. Par défaut les champs ne peuvent pas être vides (null). 
- **nickname** qui peut être vide (**nullable**) et de type **String**.
- **parents** contient une liste de **Person**. Nous l'appelerons *champ relationel* dans la documentation. 

Les noms d'entité sont *sensibles à la casse*, **Person** et  **person** définissent deux entités différentes.

Les noms de champs sont *sensibles à la casse*, un champ nommé **name** est différent de **Name** 

# Champ Scalaires
Discret definit les types scalaires suivant pour stocker vos données:

- **Integer**: Un entier 64‐bit signé,
- **Float**: Un nombre flottant 64-bit,
- **String**: Une séquence de caractères UTF‐8,
- **Boolean**: "true" ou "false" (vrai ou faux) ,
- **Json**: Une séquence de caractères UTF‐8 contenant un objet Json,
- **Base64**: Un object binaire codé en base64.

Les types scalaires sont *insensibles à la casse*. Par exemple, **integer** ou **iNteGer** sont valides.

Par défaut les champs de type scalaires *ne peuvent pas* être vide (nullable). Mais il est possible de:
- leur fournir une valeur par défaut,
- definir le champ "nullable" (pouvant être vide)
  
il n'est pas possible de combiner **nullable** and **default** pour le même champ.

```js
{   
    //avec des valeurs par défaut
    ScalarsDefault {
        name: String default "someone",
        age: Integer default 0,
        height: Float default 3.2,
        is_nice: Boolean default true,
        Is_Nice: boolean default True, //true est insensibles à la casse
        profile: Json default "{}",                
        thumbnail: Base64 default ""
    }

    //avec des valeurs pouvant être vides
    ScalarsNullable {
        name: String nullablE,
        age: Integer NULLABLE, //nullable est insensibles à la casse
        height: Float nullable,
        is_nice: Boolean nullable,
        profile: Json nullable,                 
        thumbnail: Base64 Nullable
    }
}
``` 


# Champs Relationels
Les champs relationels lient des entités entre elles, permettant de définir des modèles de données complexes.
Il y a deux types de champs relationels:
- relation *unique*: le champ ne peut référencer qu'un seul tuple au plus. 
- relation *multiples*: le champ peut référencer de nombreux tuples.

Les champs relationels peuvent être vides, et on ne peut pas leur définir une valeur par défaut.  

La syntaxe est la suivante:
```js
{
    Person {
        name: String,
        pet: Pet,           //relation unique
        parents:[Person]    //relation mutiple
    }

    Pet {
        name: String,
    }
}
```
Un tuple définit par **Person** ne peut avoir qu'un seul **Pet**, mais autant **parents** que vous le souhaitez.

# Espace de nom
Les applications complexes peuvent avoir besoin d'un grand nombre d'entités. Il est possible de séparer ces entités dans des espaces de nom afin d'améliorer la modularité de l'application. Vous devriez envisager d'utiliser cette fonctionnalité pour les gros projets.

```js
mod_one {
    Person {
        name : String ,
        surname : String,
        child : [mod_one.Person] ,
        pets: [mod_one.Pet]
    }

    Pet {
        name : String,
    }
}
{
    Person {
        surname : String ,
        parents : [Person] ,
        pets: [mod_one.Pet]
    }
}
```
Cet exemple possède deux espaces de noms: **mod_one**, et l'espace de nom par défaut. On peut noter que:
 - Les deux espaces de noms contiennent une entité **Person**
 - Les champs relationels doivent référencer l'espaces de noms de l'entité liée **mod_one.Person**
 - Les champs relationels peuvent référencer des entités situées dans d'autres espaces de noms. Dans l'espace de nom par défaut, le champ  **pet** réference **mod_one.Pet**


# Champs Système 
Chaque entité possède un ensemble de champs système dédié au bon fonctionnement de la librarie. 
Every entity have a set of system fields. Those fields can be queried like regular fields, but only **room_id** and **_binary** can be modified directly.

- **id**: stocke l'itendifiant unique du tuple
- **room_id**: identitifiant de la [Room](@/learn/access_rights/room.fr.md) qui definit les drois d'access au tuple. Vous trouverez plus d'information sur les 
- **cdate**: date de creation
- **mdate**: date de la dernière modification
- **verifying_key** : identité de l'utilisation qui a modifié le tuple en dernier
- **_json**: contient les données propre à l'entité que vous avez défini.
- **_binary**: un champs binaire que vous pouvez utiliser pour stocker des données
- **_signature**: lors d'une insertion ou modification, le champs *verifying_key* est utilisé pour verifier la signature. Cela garantie l'intégrité des données echangées par les pairs.

Ces champs peuvent être intérrogés comme les champs que vous avez defini. Mais seul **room_id** et **_binary** peuvent être modifiés directement.

# Indexes
Le système définit un ensemble d'index qui devraient être suffisant pour garantir de bonnes performances dans la majeure partie des cas d'utilisation.

Si une entité possède un très grand nombre de tuples et que des problèmes de performance se font sentir, il est possible de créer des index personalisés.
Cette fonctionalité doit être utilisée avec *prudence*, car les indexes augmentente la taille des données et ralentissent les insertions.
 
Les Indexes ne peuvent être utilisés que sur des champs scalaires et systèmes. 

Cet exemple crée un index sur le tuple **(name, id)**:
```js
{
    Person {
        name: String,
        nickname: String nullable,
        parents:[Person],
        index(name, id),
    }
}
```


# Désactiver l'indexation plein texte
Par défaut, tout les champs de type **String** sont indexés ensemble pour fournir une fonctionalité de *recherche de texte libre*. Cette fonctionalité peut être désactivée en utlisant le mot clé **no_full_text_index**.

Les indexes définit dans l'entité ne sont pas désactivés.

```js
my_data {
    Person(no_full_text_index) {
        name : String,
        index(name), //non désactivé
    }
}
```

# Mettre à jour le modèle

Pendant la durée de vie d'un logiciel, le modèle de données est appelé à evoluer. Un ensemble de règles doivent être respecter pour s'assurer que les évolution ne provoque pas de régressions.

Un changement de modèle de donné doit contenir le modèle complet, pas seulement les modifications. *Discret* va comparer l'ancien et le nouveau modèle pour s'assurer que les règles suivantes sont respectées:
- Il est interdit de supprimier des entités, champs, et espace de nom.
- Les espaces de nom doivent être insérés dans le même ordre. Les nouveaux espaces doivent être insérés à la fin.
- Les entités doivent être insérées dans le même ordre à l'interieur de leur espace de nom. Les nouvelles entités doivent être insérées à la fin.
- Les champs doivent apparaitre dans le même ordre au sein d'une entité. Le nouveaux champs doivent apparaitre à la fin.

L'insertion et la modification de champs doit obéir aux règles suivantes.
- le type d'un champ ne peut pas être modifé
- un champ scalaire peut être modifié pour devenir **nullable**
- un champs peut devenir non vide à condition de rajouter une valeur par défaut 
- un nouveau champ peut être soit *nullable* soit contenir une valeur par défaut.

Cet ensemble de contrainte empêche les regression et permet d'éviter d'avoir à gérer differentes versions du modèle de données. 

Prenons un exemple, et considéront la définition suivante comme l'ancien modèle de données.
```js
{
    Person {
        name : String nullable,
        surname: String nullable,
    }
}
```

Cette mise à jour sera acceptée:
```js
{
    Person {
        //modifié pour accepter les valeurs null
        name : String nullable, 

        //modifié pour interdire les valeur vides, en ajoutant une valeur par defaut        
        surname : String default "john",
        
        //nouveau champ à la fin de l'entité
        parents : [Person],
    }

    //nouvelle entité à la fin de l'espace de nom
    Pet {
        name: String
    }
}
//nouvel espace de nom inséré a la fin
my_data {

}
```


Mais celle-ci sera rejetée
```js
//erreur:: le nouvel espace de nom est inséré avant l'existant
my_data {

}
{
    //Erreur: nouvelle entité insérée avant l'existant
    Pet {
        name: String
    }

    Person {
        //Erreur: let champ 'name' est supprimé 

        //Erreur: nouveau champ inséré avant l'existant
        parents : [Person],

        //Erreur: champ modifié pour ne plus accepter de valeur vides,
        //mais sans ajouter de valeur par défaut.
        surname : String,
        
    }
}
```


# Dépréciation
Nous avons vu dans le précédent chapitre qu'il est interdit de supprimer des champs et des entités. Il est neanmoins possible d'indiquer aux developeurs de l'application que des champs ou entités ne doivent plus être utilisés en les "dépréciant" avec le mot clé **@deprecated**. 

C'est un mot clé de documentation, les champs et entités dépréciés peuvent toujours être utlisé et aucune erreur ou avertissement ne sera généré.

la syntaxe et la suivante:
```js
{
    @deprecated Person {
        name : String ,
    }

    Pet {
        name : String,
        @deprecated  age : Float NULLABLE,
        weight : Integer NULLABLE,
        is_vaccinated: Boolean NULLABLE,
        INDEX(weight),
    }
}
```
Dans cet exemple, l'entité **Person** et le champ **Pet.age** sont indiqués comme déprécié. 



