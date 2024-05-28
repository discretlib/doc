+++
title ="Insert and mutate data" 
weight = 3
+++
```js
mutation {
    p1: Person { 
        room_id: $room_id
        name: "Alice" 
        pets: [{name: "Truffle"}] 
    }
    
    p2: Person { 
        room_id: $room_id
        name: "Bob" 
        children: [
            {name: "Neela"}, 
            {name: "Assa"}
        ]
    }
} 
```
