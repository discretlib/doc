+++
title ="Create your datamodel" 
weight = 1
+++
```js
{
    Person {
        name: String,
        children: [Person],
        pets: [Pet],
    }

    Pet {
        name: String,
    }
}
```