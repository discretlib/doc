+++
title = "Configuration"
description = "En apprendre plus sur les options de configuration"
weight = 1
+++

*Discret* propose un certain nombre d'options de configuration qui sont passées au démarrage du système.

---

```js
parallelism: integer
```
Définit le niveau parallélisme de l'application. Ce nombre impacte:
- le nombre de **Room** qui peuvent être synchronisées en parallèle
- le nombre de *threads* de lecture de la base de données
- le nombre de *threads* de verification de signatures
- le nombre de *buffers* utilisés pour lire et écrire les données sur le réseau
- la taille des *channels* utilisés pour transmettre des messages entre les services internes

Un plus grand nombre ca fournir de meilleures performances au prix d'une plus grande utilisation mémoire. Un nombre plus grand que le nombre de CPU ne devrait pas apporter de gains importants.

---

```js
auto_accept_local_device: boolean
```
Lorsque vous vous connectez avec le même secrets sur deux appareils différents, ces deux appareils s'échangent une signature matérielle pour vérifier qu'ils sont autorisés à se connecter. 

Cela ajoute un niveau additionnel de sécurité dans le cas où votre secret serait aussi utilisé par une autre personne sur Internet. Cela peut arriver dans le cas où vos utilisateurs utilisent des mots de passe faibles. 

Si la connection se fait sur internet, un nouvel appareil sera toujours rejeté.
  
Par contre, sur un réseau local un nouvel appareil sera autorisé par défaut. Ce comportement peut être désactivé en modifiant 'auto_accept_local_device' à false.
Dans ce cas, quand un nouvel appareil est détecté sur le réseau local:
- un tuple **sys.AllowedHardware** sera créé avec le status:'pending'
- un évènement **PendingHardware** sera déclenché
- la tentative de connection sera rejetée

---

```js
auto_allow_new_peers: boolean
```
Définit le comportement de *Discret* quand il découvre un nouveau pair lors de la synchronisation des **Rooms**

auto_allow_new_peers=true:
- Je fais confiance aux amis de mes amis et ceux-ci seront autorisés à se connecter directement à mes appareils. 

auto_allow_new_peers=false:
- par défaut, les amis de mes amis ne sont pas autorisés à se connecter à mes appareils. C'est la configuration par défaut.

Par exemple, imaginons que vous avez invités Bob à discuter avec vous. Bob crée un groupe de discussion avec vous et Alice. Lors de la première synchronisation, *Discret*  va détecter un nouveau pair nommé Alice et va l'ajouter dans **sys.Peer**.

Si  auto_allow_new_peers est 'true', *Discret* va l'ajouter dans **sys.AllowedPeer** avec le status **enabled**, et l'autoriser à se connecter directement. Cela rend le réseau plus efficace car Alice pourra voir vos nouveaux messages même quand Bob ne sera pas connecté. Par contre cela se fait au prix d'une perte de vie privée car Alice connaîtra votre adresse IP. 

Si auto_allow_new_peers est 'false',
- un tuple **sys.AllowedPeer** est créé avec le status **pending**, is created in the private room, with the status set to 'pending'
- un événement **PendingPeer** est déclenché.
    
---

```js
max_object_size_in_kb: integer
```   
Définit la taille maximum d'un tuple. Les tuples doivent avoir une taille raisonnable pour garantir une synchronisation efficace.
Cet paramètre impacte la taille des *buffers* utilisés pour lire et écrire des données sur le réseau.

**/!Attention!\\** une fois votre programme en production, diminuer cette valeur cassera *Discret*. Aucune donnée ne sera perdue mais le système ne pourra plus synchroniser les tuples plus grand que cette valeur.

---

```js    
read_cache_size_in_kb: integer
```
Définit la taille maximum des caches de lecture de la base de données. Augmenter cette valeur peut améliorer les performance. Chaque *thread* de lecture va utiliser cette
quantité mémoire.

---

```js  
write_cache_size_in_kb: integer
```
Définit la taille maximum du caches d'écriture de la base de données. Augmenter cette valeur peut améliorer les performance.

---
    
```js 
write_buffer_length: integer
```
Les requêtes d'écriture en base de données sont écrite par bloc. Quand le process d'écriture est prêt, il récupère toutes les requêtes en attente jusqu'a atteindre la limite définie par ce paramètre. Ce bloc de requête sera ensuite effectué dans une unique transaction. Cela augmente énormément les performances, comparé à l'écriture des requêtes une par une (autocommit).

Pour donner une idée de l'amélioration des performance, un test d'insertion de 100 000 tuples donne le résultat suivant:
- write_buffer_length = 1       Insertions/secondes: 55  <- cela insère les données une par une
- write_buffer_length = 10      Insertions/secondes: 500
- write_buffer_length = 100     Insertions/secondes: 3000
- write_buffer_length = 1000    Insertions/secondes: 32000

Si une requête échoue au sein d'un bloc, la transaction sera annulée et toutes les autre requêtes échoueront. Cela ne devrait pas être un problèmes car les requêtes d'écriture n'échouent qu'en cas de problème matériel (plus de place sur le disque dur, par exemple) ou de *Bug*. Dans les deux cas, cela ne pose pas de problème d'annuler tout un bloc de données.

---    

```js 
announce_frequency_in_ms: integer
```
La fréquence (en millisecondes) à laquelle des annonces sont envoyés sur le réseau pour trouver vos pairs.
    
---

```js 
enable_multicast: boolean
```   
Active/désactive la découverte de vos pairs sur le réseau local

---

```js 
multicast_ipv4_interface: String
```   
*Discret* utilise la fonction *IP multicast* pour découvrir vos pairs sur le réseau local.Sur certain système avec plusieurs cartes réseau, il pourrait être nécessaire d'indiquer votre adresse IP locale pour que le *multicast* fonctionne. La valeur par défaut (laisser le système d'exploitation se débrouiller) devrait néanmoins fonctionner dans la plupart des cas. 

---

```js 
 multicast_ipv4_group: String
``` 
Le groupe *IP multicast* utilisé. 

---

```js 
pub enable_beacons: boolean
```
Active/désactive la découverte de vos pairs  sur internet.

---

```js  
pub beacons: List<BeaconConfig>,
```
Liste des serveur *Beacon* utilisés pour découvrir vos Pairs sur internet. Voici un exemple de configuration au format TOML:
```toml 
[[beacons]]
hostname = "sever.com:4264"
cert_hash = "weOsoMPwj976xqxRvLElsbb-gijWWn0netOtgPflZnk"

[[beacons]]
hostname = "192.168.1.8:4266"
cert_hash = "weOsoMPwj976xqxRvLElsbb-gijWWn0netOtgPflZnk"
```

Un serveur beacon comporte deux champs:
- *hostname*: l'adresse du serveur
- *cert_hash*: le *hash* du certificat utilisé par ce serveur. Cette valeur est générée par le serveur au démarrage.

La section [Beacon](@/learn/configuration/beacon.fr.md) fournit les informations nécessaires pour créer un serveur et récupérer son *cert_hash*.

---
    
```js      
pub enable_database_memory_security: boolean
```
Empêche la mémoire d'être écrite en *swap* et vide la mémoire(*zeroise*) quand elle est libérée.

Désactivé par défaut car cela impacte fortement les performances(d'environ 50%). Cela ne devrait être utilisé que si votre système demande un niveau de sécurité extrêmement élevé.

source: [cipher_memory_security](https://discuss.zetetic.net/t/what-is-the-purpose-of-pragma-cipher-memory-security/3953)
