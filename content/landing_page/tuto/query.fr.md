+++
title ="Interrogez les données" 
weight = 4
+++
```ts
query {
    q1: Personne(room_id=$room_id) { 
        id
        nom 
        bestioles(nom="Truffe") {
            nom
        } 
    }
    
    q2: Personne(search("bob")) { 
        id
        nom 
        enfants(order_by(nom DESC)){
            id
            nom
        }
    }
} 
```