+++
title = "Schema and Entities"
weight = 1
+++

Discret uses a schema language to describe what kind of data can be inserted and queried. This document only focus on the concepts and does not provides executable code.

# Basic Syntax 

Let's start with a basic example.
```gql
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
- **parents** contains an array of **Person**. We we call it a *relation* field in the documentation 

Entity names are *case censitive*, **Person** and  **person** would define two different entities.

Field names are *case censitive*, a field **name** will be different from **Name** 

# Scalar Fields
Discret used the following scalar type that will store your data:

- **Integer**: A signed 64‐bit integer,
- **Float**: A 64-bit floating-point value,
- **String**: A UTF‐8 character sequence,
- **Boolean**: true or false,
- **Json**: a Json String,
- **Base64**: a base64 encoded String to store binary data.

Scalar types are *case incensitive*. for example, **integer** or **iNteGer** is valid.

By default scalar type are *not* nullable. but it is possible to :
    - provide a default value
    - make the field not nullable
    - 
You cannot combine not nullable and default value for the same field.

```gql
{   
    //with default values
    ScalarsDefault {
        name: String default "someone",
        age: Integer default 0,
        height: Float default 3.2,
        is_nice: Boolean default true,
        Is_Nice: boolean default True, //check case sensitivity
        profile: Json default "{}",                
        thumbnail: Base64 default ""
    }

    //with null values
    ScalarsNullable {
        name: String nullablE,
        age: Integer NULLABLE,
        height: Float nullable,
        is_nice: Boolean nullable,
        Is_Nice: boolean nullable, 
        profile: Json nullable,                 
        thumbnail: Base64 Nullable
    }
}
``` 




# Relation Field
Relation fields link object together, allowing complex data modelisation and query.
There is two kind of relation field:
- *single* relationship where the field can reference one object at most 
- *multiple* relationship where the field can reference many objects

It is not possible to define default values for relation fields 

The syntax is the following.
```gql
{
    Person {
        name: String,
        pet: Pet,           //single relationship
        parents:[Person]    //mutiple relationship
    }

    Pet {
        name: String,
    }
}
```
Sadly, the **Person** defined in this model can only have one **Pet**. But they can have has many **parents** as you want.

# Namespace
Complex applications can have a large number of entities. It is possible to separate entity definition in *namespace*. This can greatly improve the modularity of your code and should be considered for large codebase.

```gql
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
mod_two{
    Person {
        surname : String ,
        parents : [mod_two.Person] ,
        pets: [mod_one.Pet]
    }
}

```
In this example two namespace are defined: **mod_one** and **mod_two**, we can notice that:
 - Both namespace contains a **Person** entity
 - Relation field must references the namespace of the linked entity like **mod_one.Person**
 - Relation field can references external namespace. In **mod_two**, the pet field references **mod_one.Pet**



# System fields 
Every entity have a set of system fields. Those fields can be queried like regular fields, but only **room_id** and **_bin** can be modified directly.

- **id**: the unique identifier of the object.
- **room_id**: id of the room that contains the access right for the object
- **cdate**: creation date
- **mdate**: last modification date
- **verifying_key** : identity of the user that created or last modified the object
- **_json**: stores the entity data
- **_bin**: a binary field to store anything
- **_signature**: upon insertion, the verifying_key is used to verify this signature to ensure data integrity


# Index
If an entity contains a very large number of object, and if a specific set of fields are queried a lot, it is possible to create an index to improve query performances. Putting two many indexes can result in degraded insertion performances, so you should use this feature *wisely*. The base system allready have a set of indexes that should be enought for a lot of use cases. 
 
Indexes can only be put on scalar field and system fields. 
```gql
{
    Person {
        name: String,
        nickname: String nullable,
        parents:[Person],
        index(name, id),
    }
}
```
The example creates an index on the **(name, id)** tuple


# Disabling full text indexing
By default, every String fields are indexed to perform full text query. It can be disabled using the **no_full_text_index** flag.
The index defined inside the entity will still be functional.

```gql
my_data {
    Person( no_full_text_index) {
        name : String,
    }
}
```

# Update constraints




# Deprecation
We just saw in the previous chapter that entities and fields cannot be deleted. It is however possible to mark them as 'deprecated' to indicate the developers of your application to stop using them. Deprecated field and entitie can still be used and no warning or error will be thrown.

Deprecation uses the **@deprecated** keyword:
```gql
{
    @deprecated Person {
        name : String ,
    }

    Pet {
        name : String,
        @deprecated  age : Float NULLABLE,
        weight : Integer NULLABLE,
        is_vaccinated: Boolean NULLABLE,
        INDEX(weight)
    }
}
```
In this example, the **Person** entity and the **Pet.age** field are marked deprecated. 



