+++
title ="Insérez vos données" 
weight = 3
+++
```ts
mutation {
    p1: Personne { 
        room_id: $room_id
        nom: "Alice" 
        bestioles: [{nom: "Le Chat"}] 
    }
    
    p2: Person { 
        room_id: $room_id
        nom: "Bob" 
        enfants: [
            {nom: "Neela"}, 
            {nom: "Assa"}
        ]
    }
} 
```
