+++
title ="Insérez vos données" 
weight = 3
+++
```ts
mutate {
    p1: Personne { 
        room_id: $room_id
        nom: "Alice" 
        bestioles: [{nom: "Truffe"}] 
    }
    
    p2: Personne { 
        room_id: $room_id
        nom: "Bob" 
        enfants: [
            {nom: "Neela"}, 
            {nom: "Assa"}
        ]
    }
} 
```
