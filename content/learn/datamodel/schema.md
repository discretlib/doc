+++
title = "Schema and Entities"
description = "learn how to create and update the data model"
weight = 1
+++

Discret uses a data model language to describe what kind of data can be inserted and queried.


# Basic Syntax 

Let's start with a basic example:
```js
{
    Person {
        name: String,
        nickname: String nullable,
        parents:[Person]
    }
}
```
We defined an entity named **Person** that contains three fields:
- **name** which must contains a **String**. By default, fields are not nullable 
- **nickname** which is defined as nullable and may contain a **String**
- **parents** contains an array of **Person**. We we call it a *relation field* in the documentation 

Entity names are *case censitive*, **Person** and  **person** would define two different entities.

Field names are *case censitive*, a field **name** will be different from **Name** 

# Scalar Fields
Discret defines the following scalar types to store your data:

- **Integer**: A signed 64‐bit integer,
- **Float**: A 64-bit floating-point value,
- **String**: A UTF‐8 character sequence,
- **Boolean**: true or false,
- **Json**: a Json String,
- **Base64**: a base64 encoded String to store binary data.

Scalar types are *case incensitive*. for example, **integer** or **iNteGer** is valid.

By default scalar types are *not* nullable. but it is possible to :
- provide a default value
- make the field *nullable*
  
You cannot combine **nullable** and **default** for the same field.

```js
{   
    //with default values
    ScalarsDefault {
        name: String default "someone",
        age: Integer default 0,
        height: Float default 3.2,
        is_nice: Boolean default true,
        Is_Nice: boolean default True, //true is case incensitive
        profile: Json default "{}",                
        thumbnail: Base64 default ""
    }

    //with nullable values
    ScalarsNullable {
        name: String nullablE,
        age: Integer NULLABLE, //nullable is case incensitive
        height: Float nullable,
        is_nice: Boolean nullable,
        profile: Json nullable,                 
        thumbnail: Base64 Nullable
    }
}
``` 


# Relation Field
Relation fields link entities together, allowing for complex data modelisation and query.
There is two kinds of relation field:
- *unique* relationship where the field can reference one tuple at most 
- *multiple* relationship where the field can reference many tuples

Relation fields  are nullable and it is not possible gives them a default value. 

The syntax is the following.
```js
{
    Person {
        name: String,
        pet: Pet,           //unique relationship
        parents:[Person]    //mutiple relationship
    }

    Pet {
        name: String,
    }
}
```
Sadly, the **Person** defined by this entity can only have one **Pet**. But they can have has many **parents** as you want.

# Namespace
Complex applications can have a large number of entities. It is possible to separate entity definition in *namespace*. This can greatly improve the modularity of your code and should be considered for large codebase.

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
In this example two namespace are defined: **mod_one** and the default namespace without a name, we can notice that:
 - Both namespace contains a **Person** entity
 - Relation field must references the namespace of the linked entity like **mod_one.Person**
 - Relation field can references external namespace. In the empty namespace, the pet field references **mod_one.Pet**


# System fields 
Every entity have a set of system fields, most of which being used internaly by Discret.
- **id**: the unique identifier of the tuple.
- **room_id**: id of the [Room](@/learn/access_rights/room.md) that contains the access rights for the tuple
- **cdate**: creation date
- **mdate**: last modification date
- **verifying_key** : identity of the user that created or last modified the tuple
- **_json**: stores the entity data
- **_binary**: a binary field. This field is not used internaly and can be freely used to store any data
- **_signature**: upon insertion, the verifying_key is used to verify this signature to ensure data integrity

Those fields can be queried like regular fields, but only **room_id** and **_binary** can be modified directly.

# Index
The base system already possesses indexes that should be enought for a lot of use cases. 
If an entity contains a very large number of tuples, and if a specific set of fields are queried a lot, it is possible to create an index to improve query performances. you should use this feature *wisely*, two many indexes can result in degraded insertion performances, . 
 
Indexes can only be put on scalar and system fields. 

This example creates an index on the **(name, id)** tuple:
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


# Disabling full text indexing
By default, every **String** fields are indexed to allow full text search. It can be disabled using the **no_full_text_index** flag.

The index defined inside the entity will not be disabled.

```js
my_data {
    Person( no_full_text_index) {
        name : String,
        index(name) //not disabled
    }
}
```

# Updating the datamodel

During the lifetime of a software, the datamodel is likely to be modified many times. A set of rules must be respected to ensure that adding new fields and entities will not introduce breaking changes. 

A data model modification must contains the full data model, not just the changes. Discret will compare the new model against the existing one to enforce the following rules:

- Entities, Fields and Namespaces cannot be deleted.
- Namespace must be inserted in the same order, new Namespaces must be inserted at the end.
- Entities must be inserted in the same order wihtin their namespace. New entities must be inserted at the end of a namespace.
- Fields must appear in the same order in an entity. New fields must be inserted at the end of the entity.

Fields update and insertion must obey strict rules:
- Fields type cannot be updated.
- Fields can be made *nullable*.
- Fields can be made not nullable only if a *default* value is provided. 
- New Fields must be either *nullable* or contains a default value.

This set of constraints allows the datamodel to evolve without requiring versionning.

Let take an example, if we consider the following to be the old data model 
```js
{
    Person {
        name : String nullable,
        surname: String nullable,
    }
}
```

The following update will be valid
```js
{
    Person {
        //changed to nullable
        name : String nullable, 

        //changed to not nullable with a default value        
        surname : String default "john",
        
        //new field at the end
        parents : [Person],
    }

    //new entity at the end of the namespace
    Pet {
        name: String
    }
}
//new namespace after the existing one
my_data {

}
```


But this one will be rejected
```js
//error:: new namespace before the existing one
my_data {

}
{
    //Error: new entity before an existing one
    Pet {
        name: String
    }

    Person {
        //Error: 'name' field is removed

        //Error: new field before an existing one
        parents : [Person],

        //Error: changed to not nullable without a default value
        surname : String,
        
    }
}
```



# Deprecation
We saw in the previous chapter that entities and fields cannot be deleted. It is however possible to mark them as 'deprecated' to indicate the developers of your application to stop using them. This is only a documentation flag, deprecated fields and entities can still be used and no warning or error will be thrown.

Deprecation uses the **@deprecated** keyword:
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
In this example, the **Person** entity and the **Pet.age** field are marked deprecated. 



