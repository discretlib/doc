+++
title ="Query" 
weight = 4
+++
```js
query {
    q1: Person(room_id=$room_id) { 
        id
        name 
        pets(name="Truffle") {
            name
        } 
    }
    
    q2: Person(search("bob")) { 
        id
        name 
        children(order_by(name DESC)){
            id
            name
        }
    }
} 
```