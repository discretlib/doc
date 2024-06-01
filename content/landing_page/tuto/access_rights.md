+++
title ="Define Access Rights" 
weight = 2
+++
```ts
mutate {
    sys.Room {
        admin: [{
            verif_key: $peer_key
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