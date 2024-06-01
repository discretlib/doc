+++
title ="Gérez les droits d'accès" 
weight = 2
+++
```ts
mutate {
    sys.Room {
        admin: [{
            verif_key: $peer_key
        }]

        authorisations: [{
            name: "maison"
            rights:[{
                entity: "Personne"
                mutate_self: true
                mutate_all: true
            },{
                entity: "Bestiole"
                mutate_self: true
                mutate_all: false
            }]
        }]
    }
}
```