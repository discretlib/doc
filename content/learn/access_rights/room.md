+++
title = "Rooms"
description = "Learn how to manage data access rights"
weight = 1
+++

*Discret* data synchronization uses fined grained access rights to decide which data needs to be synchronized with peers. Every tuple can be put in a *Room* that defines a set of rights:
- who can modified data
- who can read data
- what kind of entity is valid
- who can admin the *Room* 

Each tuple can be in a single *Room*, but can be moved across *Rooms*.

When inserting or mutating a tuple belonging to a *Room*, the access rights will be used to verify that the peer has the necessary rights do perform the action. 

When connecting with other peers, the *Rooms* will be used to known which data needs to be synchronized with those peers. Other peers will not receive data from *Rooms* they don't belong to.

When synchronising data, each tuple will be checked to ensure received data have the necessary rights in the *Rooms*

# Schéma

The Room data model is the following:
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

The **Room** entity is the root and contains:
- **admin**: the *Room* administors,
- **authorisations**: the access rights. 

The **Authorisation** entity contains access rights for a *Room*. A *Room* can contains several authorizations:
- **name**: a descriptive name, for example: authors, readers, etc.
- **rights**: entity level access rights: what kind of entity can be in this room,
- **users**:  the authorised users,
- **user_admin**: administrators that can add or disable users.

The **UserAuth** entity defines users:
- **verif_key**: the cryptographic signature verifying key of the user,
- **enabled**: enable/disable this user,
- **mdate**: the **mdate** system field defines the validity starting date of this user authorization.  


Every users defined in a *Room* have a read only access on all data. The **EntityRight** entity defines the mutation rights for an entity:
- **entity**: the entity name, including its namespace. For example: **house.Person**. 
- **mutate_self**: defines the right to insert tuples and modify data you have inserted 
- **mutate_all**: defines the right to modify data created by other users.
- **mdate**: the **mdate** system field defines the validity starting date of this access right.

while not recommended, it is possible to global mutation rights on all entities by setting the **\*** wildcard in the **entity** field. Wilcard right will only be used if an inserted entity is not defined in a **EntityRight**.

To guarantee data integrity, some *Room* modifications constraint are enforced:
- it is forbidden to delete a Room sub entity. for example **Authorization** or **EntityRight** cannot be deleted.
- it is forbidden de modify the **UserAuth** et **EntityRight** tuples. if you need to modify an existing right, you will need to create a new version.

# Inseting data in a *Room*

Every tuple have a **room_id** system field that references a *Room* identifier.

For example, if **$room_id** is the identifier of an existing *Room*, the following query will insert a  **Person** tuple in the *Room* 
```js
mutate {
    Person{
        room_id: $room_id
        name: "Alice"
    }
}
```
During the creation process, the tupple will be signed with your cryptographic signature key and your verifying key will be inserted in the **verifying_key** system field.

The tuple signature and an verifying key will then be used to ensure that you have the right to insert this data in the *Room*.

# Example: access rights for a *Blog* 

To create a blog, we will use this simplified data model. 
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
The blog author must have the following rights
- insert and modify **Articles**
- insert and modify **Comment**
- modify **Comment** from other user, for moderation purpose.

A blog reader must have the following rights
- read **Articles**
- insert and modify **Comment**

The blog *Room* will be created using the following query:
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

Lines *7* to *23* create authorizations for the authors, and line *24* to *34* create authorizations for readers.

You can notice that:
- not rights have been defined for the **Article** entity for the readers. Every users having an access to the *Room* can read every entities.
- no **user_admin** have been defined for both authorizations. It means that only **admin** of the *Room* can manage users.


# Example: a shared calendar
This example is more complex than the previous one and show interactions between several *Rooms*. 

We want to have two different access rights between an appointment dates and its details. For example: 
- we want to share our calendar with every employee of our company but without revealing details about appointments
- we want to share our calendar with the details to our team.

As every **Room** users have access to every data, we need to define two *Rooms*:
- one to store appointment dates. It will be named **$room_calendar**,
- one to store appointment details.It will be named **$room_cal_detail**

We will use this simplified data model:
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

The following users will be used:
- **$author**: the calendar owner
- **$team_user**: user that have access to the detail
- **$collaborator**:  user that can only see appointment dates

The **$room_calendar** *Room* is created using the query:
```js
mutate {
    sys.Room{
        admin: [{
            verif_key:$author
        }]
        authorisations:[{
            name:"authors"
            rights:[
                {
                    entity:"cal.Calendar"
                    mutate_self:true
                    mutate_all:true
                },
                {
                    entity:"cal.Appointment"
                    mutate_self:true
                    mutate_all:true
                }
            ]
            users:[{
                verif_key:$author
            }]
        },{
            name:"readers"
            users:[{
                verif_key:$team_user
            },{
                verif_key:$collaborator
            }]
        }]
    }
}
```
Users that are not the author don't have any mutation rights, so no **rights** are defined for the **readers** authorization.

The **$room_cal_detail** *Room* is created using the query:
```js
mutate {
    sys.Room{
        admin: [{
            verif_key:$author
        }]
        authorisations:[{
            name:"authors"
            rights:[
                {
                    entity:"cal.AppointmentDetail"
                    mutate_self:true
                    mutate_all:true
                }
            ]
            users:[{
                verif_key:$author
            }]
        },{
            name:"readers"
            users:[{
                verif_key:$team_user
            }]
        }]
    }
}
```
You will notice that **$team_user** is the only one allowed in the **readers** authorization.

A new calendar will be created with the following query:
```js
mutate {
    res: cal.Calendar{
        room_id: $room_calendar
        name: "my calendar"
    }
}
```

For a calendar with an **id** set to **$calendar_id**, a new appointment will be inserted with the following query: 
```js
mutate {
    cal.Calendar {
        id: $calendar_id
        appointments:[{
            room_id: $room_calendar_id
            start:  1720770000000
            end:    1720780000000
            detail:{
                room_id: $room_cal_detail_id
                title: "An important meeting"
            }
        }]
    }
}
```

The following query will retrieve the appointments:
```js
query {
    res: cal.Calendar(
        id=$calendar_id
    ) {
        name
        appointments(
            nullable(detail)
            ){
            start
            end
            detail {
                title
            }
        }
    }
}
```

The **$team_user** user will get the following result:
```json
{
    "res":[{
        "name":"my calendar",
        "appointments":[{
            "start":1720770000000,
            "end":1720780000000,
            "detail":{
                "title":"An important meeting"
            }
        }]
    }]
}
```

The **$collaborator** user who cannot access the details wil get the following result:
```json
{
    "res":[{
        "name":"my calendar",
        "appointments":[{
            "start":1720770000000,
            "end":1720780000000,
            "detail":null
        }]
    }]
}
```