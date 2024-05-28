+++
title ="Define Access Rights" 
weight = 2
+++
```js
mutation {
    sys.Room {
        admin: [{
            verif_key: $user_id
        }]

        authorisations: [{
            name: "house"
            rights:[{
                entity: "Person"
                mutate_self: true
                mutate_all: true
            },{
                entity: "Pet"
                mutate_self: true
                mutate_all: false
            }]
        }]
    }
}
```