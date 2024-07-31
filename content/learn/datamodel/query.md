+++
title = "Queries"
description = "learn how to use the query engine and all its options"
weight = 3
+++

*Discret* provides a query engine inspired by GraphQL

Most of the examples of this document will use the following [data model](@/learn/datamodel/schema.md):
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

with the following data:
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


# Base Syntax

Let's start with a simple query that will retrieve the name and surname of the **Person**:
```js
query {
    result: Person {
        name
        surname
    }
}
```

You will notice that:
- **result** is an alias, it can have a different name,
- the requested fields *are not* delimited by a comma.

This will return the following JSON object:
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
You will notice that:
- the query result is a JSON table, this is true even if there is only one result,
- If the database does not contains requested data, the query will return an empty table.

# Multiple Queries
You can perform several queries at the same time. The following query will return the **Pet** list and the **Person** list.
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

**persons** et **pets**  aliases must be unique withing the query because each one will generate a JSON object field.

It will return the following JSON object:
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



# Filters
Previous examples recovers all the tuples of a specified entity, in real life most query will need to filter data.

Filters supports the following operators:
- **=**     equals
- **!=**    not equals
- **\>**    greater than
- **\>=**   greater than or equals
- **<**     lower than
- **<=**    lower than or equals


The syntax is the following:
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
This **Person** query is filtered by name and indicates that we only want to retrieve tuples whose name equals to "Doe". It provides the following result: 
```json
{
    "Person":[
        {"name":"Doe","surname":"John"},
        {"name":"Doe","surname":null}
    ]
}
```

You can define multiple filters by separating them with a comma. 
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

You can also filter using the **null** value:
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


# Full text search
By default, every text field is indexed to allow full text search using the **search** keyword.
The search engine uses a *trigram* index, only string greater or equal than 3 characters can be searched.

The indexing can be [disabled for an entity](@/learn/datamodel/schema.md#disabling-full-text-indexing).


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
This will only returns the "Alice" tuple:
```json
{
"Person":[{"name":"Coop","surname":"Alice"}]
}
```

# Nested queries
You can query an entity and its relations at the same time.

The following query get the **Person** and their **Pets**:
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
It produces the following result:
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

The query only provides **Person** that have at least a **Pet**. **Person** without pets are filtered. 
This makes the result very easy to use as it is guaranteed that you will never encounter a **null** value

Filters can be applied to sub-entities:
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
It produces:
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

# "Nullables" Relations

For some query, you may wan't to recover entities with empty relations.It can be done by using the **nullable** keyword that defines the relation fields that can be null for the query. For example:


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
For this query, the **pet** field is allowed to be null and the query will return all the **Person**:

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

Once a field has been defined  **nullable**, you can add a filter to only get tuples that have an empty relation:

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

# Sorting result
When a query returns several tuples, the tuple order is not guaranteed. Sorting results can be done by using the **order_by** keyword.

You can sort on multiple fields. Each field must have a sort order:
- **asc** lower to greater (ascending)
- **desc** greater to lower (descending)

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

The query returns:
```json
{
    "Person":[
        {"name":"Doe","surname":null},
        {"name":"Doe","surname":"John"},
        {"name":"Coop","surname":"Alice"}
    ]
}
```



The **order_by** clause can be used in sub-entities.

# Limiting the number of results

If a query is expected to return a large number of results, you can limit the number of field by using the  **first** keyword.

The following query returns the first two tuples:
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
```json
{
    "Person":[
        {"name":"Doe","surname":null},
        {"name":"Doe","surname":"John"}
    ]
}
```

You can also skip a number of tuples by using the **skip** keyword.

The following query skip the first result and returns the next two tuples.
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

```json
{
    "Person":[
        {"name":"Doe","surname":"John"},
        {"name":"Coop","surname":"Alice"}
    ]
}
```


# Pagination
It is tempting to use the **skip** and **first** keyword to implement pagination for large set of data, but this method can have performance issue because skipping tuple can be expensive for large dataset.

*Discret* provides an alternative with the **before** et **after** keywords. Those keyword work in association with the **order_by** an allow to filter the sorted fields.

The query syntax is the following:
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
The returned tuples will match those rules 
-  **mdate** system field greater than the **\$date** parameter
-  if the **mdate** is equal to **\$date**, it will ensure that the **id** system field is greater than the **\$id** parameter.

This method is much faster than the **skip** and **first** one.


# Json Selector

JSON fields can be explored using the JSON selector **->$.**.

Let's consider the following data model:
```js
{
    Article {
        content: json
    }
}
```

with the following tuple 
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

The following query will recover the title of the article
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

# Aggregates

*Discret* provides the following aggregate function. **(field)** is the name of the field to aggregate.
- **avg(field)**
- **count()**
- **max(field)**
- **min(field)**
- **sum(field)**

An aggreation field has the form **alias: function()**. In a query, if an entity defines an aggregated field, all other fields must also be aggregated.


The following query counts the number of parents for a **Person**:
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
and provides the following result:
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
We can see that the **parents** field is used two times:
- once with the alias **parent_count** to count the number of parents
- once to recover the parents name

It is mandatory to use it two times because we cannot define aggregates and normal field in the same entity. The following query with **name** et **count()** would make no sense and will return an error:
```js
    query {
        Person (
            nullable(parents)
        )
        {
            name
            //this sub-entity query does not make sense
            parents {
                total: count()
                name
            }
        }
    }
```