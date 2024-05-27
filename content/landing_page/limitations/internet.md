+++
weight = 4
+++

For peer to peer connection over the Internet, a discovery server is needed to allow peers to discover each others. The Discret library provides an implementation of the discovery server named Beacon.

However, connection over the internet is not 100% guaranted to work, because certain types of enterprise firewalls will block the connection attempts. Implementing a relay server could fix the issue, but it is in not planned yet.