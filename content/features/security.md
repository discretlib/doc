+++
title="Strong Security"
weight = 2
+++

The library provides strong security features out of the box: 
- data is encrypted at rest by using the SQLCipher database 
- encrypted communication using the QUIC protocol 
- data integrity: each rows is signed with the peer signing key, making it very hard to synchronise bad data 
- access control via Rooms