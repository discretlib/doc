+++
title = "System Events"
description = "Learn more about system events triggered by Discret"
weight = 1
+++

*Discret* react to some actions by triggering system wide events. Those events can be listened by using the API function **subscribe_for_events**.

# DataChanged(data_modification)

This event is triggered whenever data is modified or inserted. Data is inserted/deleted in batches and this events describes each batch.

**data_modification** contains a *HashMap*:
- the key is the identifier of the *Rooms* that have been modified
- the data contains the modified Entity name and the mutation days (date without hour:minutes:second).
 
```rust
DataModification {
    rooms: HashMap<room_id, HashMap<entity_name, Vec<date>>>,
}
```

# RoomModified(room)
This event is triggered when a *Room* is modified.

**room** contains the modified *Room* in the Rust API. For the Flutter API, only the *Room* is returned.

# PeerConnected(verifying_key, date, connection_id)

This event is triggered when a peer has connected successfully to your device.
- **verifying_key**: the peer verifying key,
- **date**: the connection date,
- **connection_id**: the unique identifier of the connection

# PeerDisconnected(verifying_key, date, connection_id)

This event is triggered when a peer have been disconnected 
- **verifying_key**: the peer verifying key,
- **date**: the connection date,
- **connection_id**: the unique identifier of the connection

# RoomSynchronized(room_id)

This event is triggered when a *Room* has been synchronized.
- **room_id**: the *Room* identifier

# PendingPeer
This event is triggered when a new peer is found when synchronizing a **Room**.
A list of pending peers can be retrieved using the following query:

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
This event is triggered when a new device is detected. 
A list of devices waiting for manual authorization can be retrieved using the following query:

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
