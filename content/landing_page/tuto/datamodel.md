+++
title ="Create your datamodel" 
weight = 1
+++
```ts
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