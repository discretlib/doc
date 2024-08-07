+++
title = "The Beacon Server"
description = "Learn how to install and configure a meeting server"
weight = 3
+++

On a local network, **Discret** can discover other peers without any specific hardware. But when peers are connecting over Internet a meeting server named beacon is required to allow peers to discover each others.

The beacon server works as follow. Let's suppose that Alice and Bob want's to connect:
- Alice sends a token known by Bog to the beacons server. The sever will keep this token until Alice disconnect herself.
- Bob send the same token to the server.
- The beacon server acknowledge that Alice and Bob share the same token:
  - it sends Alice's public IP address to Bob
  - And send Bob'as public IP address to Alice
- Alice and Bob know their respective public address and can initiate a connection.
- Once connected Alice and Bob check their identity, because the shared token is not a proof of identity  but just a mean to find each others

# Hosting

The Beacon server is used to resolve public IP address of peers that want's to connect. It cannot be installed on your local local network because it will fail to resolve your public IP. It must be hosted elsewhere. 

The server is not ressource intensive and can be hosted a cheap server as a start. For example:
- a free tier server from a cloud company,
- a cheap virtual private server (VPS).
- etc.

# Installation

No pre-compiled binaries are available, you have to compile it yourself.

Starts by cloning the beacon repository:
```
git clone https://github.com/discretlib/discret_beacon.git
```

and then compile it, the compilation can take a few minutes:
```
cd discret_beacon
cargo build --release
```
The compiled server will be located in the  *./target/release/*. Its exact name depends on you OS, on Linux it is named  **discret_beacon**.

Copy the file in a folder of your choice, then run it. On Linux, the following command will start it as a background task:
```
nohup ./discret_beacon &
```

# Usage
At first run, *Beacon* will create the following file and folders:

```
logs/
Beacon.conf.toml  
certificate_hash.txt  
cert_der.bin
log4rs.yml
```

- **logs/**: the folder that contains the application logs.
- **Beacon.conf.toml**: the **Beacon** configuraiton file that allows to change the IPV4 and IPV6 listening port.
- **certificate_hash.txt**: a text file that contains the server's certificate hash. This hash is used to register the Beacon sever in the Discret [configuration](@/learn/configuration/configuration.md) 
- **cert_der.bin**: the server certificate  and private key.
- **log4rs.yml**: The [log4rs](https://docs.rs/log4rs/latest/log4rs/) configuration file.

# Backup and Restore

The **cert_der.bin** file should be backed up to be able to reconfigure a server in case of a crash.

If you need to recover after a crash:
- copy the backed up **cert_der.bin** in the **discret_beacon** folder,
- run the server.


# Certificate Reuse
The **cert_der.bin** file can be used on several servers. For a production system, you will need to declare several Beacon server for failover.
To simplify deployment, you can use a single **cert_der.bin** for all the server which makes certificate management easier.


# Discret configuration
Let's suppose you have created three servers (firstbeacon.com, secondbeacon.com, thirdbeacon.com) with the same **cert_der.bin**.

A **Discret** client will be [configured](@/learn/configuration/configuration.md) as follow (we suppose that the configuration is stored in a TOML file):
The **cert_hash** is the value stored in the **certificate_hash.txt** file.

```toml 
beacons =[
  {hostname = "firstbeacon.com:4264",       cert_hash = "weOsoMPwj976xqxRvLElsbb-gijWWn0netOtgPflZnk"},
  {hostname = "secondbeacon.com:4264",      cert_hash = "weOsoMPwj976xqxRvLElsbb-gijWWn0netOtgPflZnk"},
  {hostname = "thirdbeacon.com:4264",       cert_hash = "weOsoMPwj976xqxRvLElsbb-gijWWn0netOtgPflZnk"},
]
```


# Beacon.conf.toml Configuration

This file contains only two parameters:
- **port**: the IPV4 listening port,



