+++
title = "Invitations"
description = "Apprenez à envoyer des invitations à d'autres pairs"
weight = 3
+++

Les invitations sont gérées dans deux entités systèmes:
```js
sys{
    OwnedInvite{
        room: Base64 nullable,
        authorisation: Base64 nullable,
    }

    Invite{
        invite_id: Base64,
        application : String,
        invite_sign: Base64,
    }
```

**OwnedInvite** contient les invitations que vous avez crées:
- **room**: indique la *Room* par défaut dans laquelle le pair sera ajouté quand il aura accepté l'invitation
- **authorisation**: indique l’autorisation de la *Room* qui sera utilisée

**Invite** contient les invitations que vous avez accepté et contient les informations nécessaires à la vérification du Pair lors de la première connection.

Vous pouvez supprimer des tuples de ces deux entités si vous voulez annuler des invitations.
