+++
title ="Créez votre modèle de données" 
weight = 1
+++
```ts
{
    Personne {
        nom: String,
        enfants: [Personne],
        bestioles: [Bestiole],
    }

    Bestiole {
        name: String,
    }
}
```