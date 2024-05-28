+++
title="Strong Security"
weight = 2
+++

Discret provides strong security features out of the box: 
- data is encrypted at rest by using the [SQLCipher](https://www.zetetic.net/sqlcipher/)  database 
- encrypted communication using the [QUIC](https://quicwg.org/) protocol 
- data integrity: data is signed with the peer signing key, making it very hard to synchronize bad data 
- access control via Rooms