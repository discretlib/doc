+++
title = "Configuration"
description = "Learn more about the configuration options"
weight = 1
+++


*Discret* provides some configuration options that will be passed to the engine at startup. 

```js
parallelism: integer
```
Defines the global parellism capabilities. This number impact:
- the maximum number of room that can be synchronized in parallel,
- the number of database readings threads
- the number of signature verification threads
- the number of shared buffers used for reading and writing data on the network
- the depth of the channels that are used to transmit message accross services

Larger numbers will provides better performances at the cost of more memory usage. Having a number larger that the number of CPU might not provides increased performances

```js
auto_accept_local_device: boolean
```
When connecting with the same key_material on different devices, thoses devices exchanges their hardaware fingerprint to check wether they are allowed to connect.
This add an extra layer of security in the unlucky case where your secret material is shared by another person on the internet (which could be relatively frequent as users tends use weak passwords).
    
When connecting over the internet, new harware is allways silently rejected.
    
However, on local network we trust new hardware by default. This behaviors can be disabled by setting 'auto_accept_local_device' to false.
In this case, when a new device is detected on the local network:
- a sys.AllowedHardware will be created with the status:'pending'
- a PendingHardware Event will be triggered
- the current coonection attempt will be rejected


```js
auto_allow_new_peers: boolean
```
Defines the behavior of the system when it discover a new peer while synchronizing a room.

auto_allow_new_peers=true:
- I implicitely trust friends of my friends. It is easy to setup, but could cause problems.

auto_allow_new_peers=false:
- Trust is given on a case by case basis, this is the recommended configuration.

Let's imagine that you have manually invited Bob to chat with you. Bob want's you to meet Alice and creates a group chat with both of you. During the synchronisation, you device detects a new peer(Alice), and add it to the **sys.Peer** list.

If auto_allow_new_peers is set to 'true', you're device will allow Alice to directly connect with you. It makes the network stronger, as Alice will be able to see your message even if Bob is not connected. But it comes at the cost of some privacy, because you now share your IP adress with Alice. In case of large communities, this setup will make your allowed peers very large, increasing the number of network connections, and increase ressources usage.

If auto_allow_new_peers is set to 'false',
- a **sys.AllowedPeer** tuple is created with the status set to **pending**
- a **PendingPeer** event is triggered
 

```js
max_object_size_in_kb: integer
```   

Defines the maximum size of an entity tupple. Object size should be kept relatively small to ensure efficient synchronisation.

This parameter has a direct impact on the size of the buffers used to read and write data on the network
Increasing this value will increase the RAM usage of the application

**!!WARNING!!** once your program is in production, decreasing this value will break the system. No data will be lost but the system will not be able to synchronized objects that are larger than the reduced value.

    
```js    
read_cache_size_in_kb: integer
```
Set the maximum cache size for the database reading threads. Increasing it can improve performances. Every read threads consumes up to that amount, meaning that increasing the "parallelism" configuration will increase the memory usage.
       
```js  
write_cache_size_in_kb: integer
```
Set the maximum of cache size for the database writing thread. increasing it may improvee performances
    
```js 
write_buffer_length: integer
```
Write queries are buffered while the database thread is working. When the database thread is ready, the buffer is sent and is processed in one single transaction.
It greatly increase insertion and update rate, compared to autocommit. To get an idea of the perforance difference, a very simple benchmak on a laptop with 100 000 insertions gives:
- Buffer size: 1      Insert/seconds: 55  <- this is equivalent to autocommit
- Buffer size: 10     Insert/seconds: 500
- Buffer size: 100    Insert/seconds: 3000
- Buffer size: 1000   Insert/seconds: 32000

If one a buffered query fails, the transaction will be rolled back and every other queries in the buffer will fail too. This should not be an issue as INSERT query are not expected to fail. The only reasons to fail an insertion are a bugs or a system failure (like no more space available on disk), and in both case, it is ok to fail the last insertions batch.
    

```js 
announce_frequency_in_ms: integer
```
how often (in milliseconds) an annouces are sent over the network to meet peers to connect to.
    
    
```js 
enable_multicast: boolean
```   
Enable/disable multicast discovery, i.e. local network peer discovery

```js 
multicast_ipv4_interface: String
```   
*Discret* uses the IP multicast feature to discover peers on local networks. On systems with multiple network interfaces, it might be necessary to provide the right ip adress for multicast to work properly. the default (let the OS choose for you) should work on most cases.
    
```js 
 multicast_ipv4_group: String
``` 
The multicast group that is used to perform peer discovery
    
```js 
pub enable_beacons: boolean
```
Enable/Disable beacon peer discovery. I.e. meet peers over the Internet.
     
```js  
pub beacons: List<BeaconConfig>,
```
List of Beacon servers that are used for peer discovery.
    
```js      
pub enable_database_memory_security: boolean
```
Enable_memory_security: Prevents memory to be written into swap and zeroise memory after free.

Disabled by default because of a huge performance impact (about 50%). It should only be used if you're system requires a "paranoid" level of security.
When this feature is disabled, locking/unlocking of the memory address only occur for the internal SQLCipher
data structures used to store key material, and cryptographic structures.

source: [cipher_memory_security](https://discuss.zetetic.net/t/what-is-the-purpose-of-pragma-cipher-memory-security/3953)