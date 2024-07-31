+++
title = "Peer Management"
description = "Learn how to list your known peers and manage connection rights"
weight = 2
+++

*Discret* maintains a list of peers that contains the known users. Those peers come from two sources:
- Peers you have invited
- Peers who have *Rooms* in common with you. For example, imagine that you have manually invited Bob to chat with you. Bob want's you to meet Alice and creates a group chat with both of you. During the synchronization, you device detects a new peer(Alice), and add it to the list.

The data model is the following:
```js
sys{
    Peer {
        pub_key: Base64 ,
        name: String default "anonymous",
    }
}
```
 **name** is defined by the peer and is not unique. It is up to the developer of an application to provide a better peer description.

**pub_key** is the peer signing key that is be used to manage its rights in a *Room* 

By default, peers you have invited are the only ones that are allowed to connect to your devices. The list of peer allowed to connect is managed with the entity:
```js
sys{
    AllowedPeer{
        peer: sys.Peer,
        meeting_token: Base64,
        last_connection: Integer default 0,
        status: String,
    }
}
```

Peers that can connect to your devices have their **status** set to **enabled**. You can forbid connection to a peer by setting its **status** to **disabled**.
