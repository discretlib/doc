+++
title = "Identifiants d'appareil"
description = "En apprendre plus sur identifiants matériels"
weight = 2
+++

Lorsque vous vous connectez avec le même secrets sur deux appareils différents, ces deux appareils s'échangent une signature matérielle pour vérifier qu'ils sont authorisés à se connecter. 

Cela ajoute un niveau additionel de sécurité dans le cas où votre secret serait aussi utilisé par une autre personne sur Internet. Cela peut arriver dans le cas où vos utlisateur utilisent des mots de passe faibles. 

Ces clés matérielles sont stockées dans l'entité système: 
```js
sys{
    AllowedHardware{
        name: String,
        status: String, 
    }
}
```

**name** décrit l'appareil authorisé.

**status** peut contenir trois valeurs:
- **enabled** : la connection à ce matériel est authorisée, 
- **disabled**: la connection à ce matériel est interdite, 
- **pending**: ce matériel est en attente d'authorisation: l'utilisateur doit manuellement authoriser ou rejeter cette clé matérielle, 

