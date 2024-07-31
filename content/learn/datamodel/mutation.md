+++
title = "Mutations and Deletions"
description = "Learn how to insert, update and delete data"
weight = 2
+++

*Mutation* queries allows for inserting and modifying data.

Examples of this document will use the following [data model](@/learn/datamodel/schema.md):
```js
{
    Person {
        name: String,
        surname: String nullable,
        parents:[Person]
    }
}
```

# Inserting data
Inserting a new tuple is done using the following query::
```js
mutate {
    Person {
        name : "Doe"
        surname: "John"
    }
}
```

This query insert a new **Person** named 'John Doe'

During the insertion, an unique identifier is stored in the **id** system field. The mutation query returns a JSON object that contains the field you have inserted and the new **id**:
```js
{
    "Person":{
        "id":"aVBDUWlHpWz9bv_vU-Feow",
        "name":"Doe",
        "surname":"John"
    }
}
```


# Multiples Mutation

A single query can performs multiple mutation:
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
 Aliases *p1, p2, last* can be any string as long as they are unique within the query as each alias will generate a JSON field in the result. The query result will be:

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
You will notice that for technical reasons, the JSON field order is not guaranteed ot be the same as the aliases order in query.

# Nesting mutation

In one single query, you can create an entity tuple with all its relations. 
The following query insert 'John Doe' and its parents.
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

You can notice that 
- Alice and Bob are not defined as **Person** because it is implied by the data model definition
- the two parents definition is separated by a comma

If some tuples have already been created, you can use them during the creation of a new tuple.
```js
mutate {
    Person {
        name : "Doe"
        surname: "John"
        parents: [
            {
                id : $mother_id //does not create a new tuple 
                                //and use the tuple references by this id
            },{
                id : "JoAWR7-zcUR5ri_yfZaqXQ" //does not create a new tuple 
                                //and use the tuple references by this id
            }
        ]
    }
}
```

# Updating Data

Updating data requires a **mutation** query that contains the *id* of the tuple to be modified. For example:
```js
mutate {
    Person {
        id: $id
        surname: "Alice"
    }
}
```
This query will update the **surname** field of the tuple with the provided **id**. if it does not exists, an error is returned.

You can add a relation to an existing tuple:
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
This query will add to the tuple with the provided **$id**:
- a reference to the tuple defined **$mother_id**
- create a new tuple **Person** for "Bob" and add its reference to the parent tuple.

You can also delete all references of a field by setting its value to **null**:
```js
mutate {
    Person {
        id: $id
        parents: null
    }
}
```


# Deleting data

Deleting data is done by using a **delete** query. You can delete a tuple or a reference in a relation field.

The following query deletes the tuple with the provided **$id**:
```js 
delete {
    Person { $id }
}
```

The following query only deletes the **parent** with the provided id **$parent _id**
```js 
delete {
    Person { 
        $id 
        parents[$parent _id]
    }
}
```


# *Room* propagation

Most of the data will be inserted in a [Room](@/learn/access_rights/room.md) using the **room_id** system field. In the case of a nested insertion, the **room_id** of the parent tuple will be propagated to its *child* tuples if no **room_id** are provided for them.


In the following example, "Alice" will be inserted in the *Room* whose id is **$room_id**.
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