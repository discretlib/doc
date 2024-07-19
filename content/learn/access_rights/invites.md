+++
title = "Invitations"
description = "learn how to invite other peers and manage invitations"
weight = 3
+++

Invites are managed by two system entities:
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

**OwnedInvite** contains invites you have created:
- **room**: is the default *Room* in which the new peer will be added when he have accepted the invite.
- **authorisation**: is the *Room* authorisation that will be used
- 
**Invite** contains the invites that you have accepted and contains the information that will allow you to verifity the peer valididy during the first connection.

You can delete tuples from those entities to cancel invites.