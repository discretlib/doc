+++
title = "Evènements système"
description = "Apprenez à utliser les évènements déclenchés par Discret"
weight = 2
+++

Discret réagit à certaines actions en envoyants des évènements systèmes. Ces évènements peuvent êre récupéré par le developper en utilisant la fonction **subscribe_for_events** de l'API

# DataChanged(data_modification)

Cet évenement est déclenché quand des données sont modifiée. Les données sont écrites par bloc et cet évènement décrit le bloc de données modifiées.

**data_modification** contient une *HashMap* ayant pour clé l'identifiant des *Room* modifiées et comme données les entités modifiées ainsi que le jour (sans les heures, minutes, secondes)  de modification. 
```rust
DataModification {
    rooms: HashMap<room_id, HashMap<entity_name, Vec<date>>>,
}
```

# RoomModified(room)
Cet évenement est déclenché quand une *Room* est modifiéé.
**room** contient la *Room* modifiée dans l'API rust. Dans l'API Flutter, seule l'identifiant de la *Room* est renvoyé.


# PeerConnected(verifying_key, date, connection_id)
Cet évenement est déclenché quand un pair se connecte.
- **verifying_key**: la clé de signature du pair
- **date**: la date de connection
- **connection_id**: l'identifiant unique de la connection

# PeerDisconnected(verifying_key, date, connection_id)
Cet évenement est déclenché quand un pair est déconnecté
- **verifying_key**: la clé de signature du pair
- **date**: la date de connection
- **connection_id**: l'identifiant unique de la connection

# RoomSynchronized(room_id)
Cet évenement est déclenché quand une *Room* a terminée une synchronisation avec un autre Pair.
**room_id** contient l'id de la *Room* synchronisée.

# PendingPeer
Cet évenement est déclenché quand un nouveau pair est detécté lors de la synchronisation d'un **Room**
la liste des pairs en attente peut être récupéré avec la requête suivante

```js
query{
    sys.AllowedPeer(
            status="pending",
            order_by(mdate desc)
        ){
         peer{
            pub_key
            name
         }
    }
}
```

# PendingHardware
Cet évenement est déclenché quand un nouvel appareil est détecté. La liste des matériels en attente d'authorisation peut être récupéré avec la requête suivante:

```js
query{
     AllowedHardware( 
            status="pending",
            order_by(mdate desc)
        ){
        name: String
    }
}
```
