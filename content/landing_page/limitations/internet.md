+++
weight = 2
+++

For peer to peer connections over the Internet, a discovery server is needed to allow peers to discover each others. *Discret* provides an implementation of the discovery server named Beacon.

However, connections over the internet is not 100% guaranteed to work, because certain types of enterprise firewalls will block the connection attempts. Implementing a relay server would fix the issue, but it is in not planned yet.