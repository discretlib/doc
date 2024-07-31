+++
title = "Le serveur Beacon"
description = "Apprenez à installer et configurer le serveur de rencontre Beacon"
weight = 3
+++

En réseau local, *Discret* peut découvrir et se connecter aux pairs sans équipement particulier. 
Mais quand les Pair sont sur Internet, un serveur de *rencontre* nommé *Beacon* est nécessaire pour que les Pairs puissent se retrouver.

Le fonctionnement du serveur de rencontre est le suivant. Supposons qu'Alice et Bob veuillent se connecter:
- Alice envoie un identifiant connu par Bob au serveur Beacon. Le serveur conserve cette information tant qu'Alice est connectée.
- Bob se connecte un peut plus tard, et envoie le même identifiant au serveur.
- Le serveur Beacon reconnaît que Alice et Bob ont envoyé le même identifiant:
  - il envoie à Alice l’adresse de Bob,
  - et envoie a Bob l'adresse d'Alice.
- Alice et Bob connaissent maintenant leur adresses publique respectives et peuvent se connecter.
- Une fois connectés, Alice et Bob vérifient leur identités respectives, car l'identifiant envoyé au serveur n'est pas une preuve d'identité mais juste un moyen de se retrouver.

# Hébergement

Le serveur Beacon a pour but de résoudre les adresses IP publiques des Pairs voulant se connecter. 

Il ne peut pas être installé sur votre réseau local car il ne pourrait pas résoudre votre adresse IP publique. Il faut donc l'héberger à l'extérieur.

Le serveur Beacon consomme peu de ressources et peut donc être installé sur un serveur peu puissant pour commencer. Par exemple:
- un serveur d'une offre gratuite cloud,
- a serveurs privés virtuels (VPS) d'entrée de game,
- etc.

# Installation

Aucune version pré-compilée n'est actuellement fournie, il vous faut donc compiler vous même le serveur.

Commencer par récupérer le projet
```
git clone https://github.com/discretlib/discret_beacon.git
```

pour ensuite le compiler, la compilation peut prendre quelques minutes:
```
cd discret_beacon
cargo build --release
```

Le programme compilé se trouvera dans le répertoire *./target/release/*. Le nom exact depend de votre système d'exploitation. 
Sous Linux le fichier se nomme *discret_beacon*;

Copiez le dans le repertoire de votre choix, puis lancez le. Sous Linux, la commande suivante le lancera en tache de fond.
```
nohup ./discret_beacon &
```

# Utilisation
Au premier lancement, beacon va créer les fichiers et répertoires suivant:

```
logs/
Beacon.conf.toml  
certificate_hash.txt  
cert_der.bin
log4rs.yml
```

- **logs/**: Le répertoire contenant les logs de l'application
- **Beacon.conf.toml**: le fichier de configuration de l'application permettant de changer le port d'écoute IPV4 et IPV6.
- **certificate_hash.txt**: un fichier contenant le *hash* du certificat utilisé par le serveur. Ce hash est utilisé dans la [configuration](@/learn/configuration/configuration.fr.md) de *Discret* pour accéder à ce serveur. 
- **cert_der.bin**: le certificat et la clé privée du serveur.
- **log4rs.yml**: le fichier de configuration des logs [log4rs](https://docs.rs/log4rs/latest/log4rs/)

# Sauvegarde et Restauration

Le fichier **cert_der.bin** devrait être sauvegarder pour pouvoir reconfigurer un serveur avec le même certificat en cas de panne.

Si vous devez reconfigurer un serveur en cas de crash:
- copiez le fichier **cert_der.bin** précédemment sauvegardé dans le répertoire de **discret_beacon**
- lancez le serveur


# Réutilisation d'un certificat
Le fichier **cert_der.bin** peut être utilisé par plusieurs serveurs.
Pour un système en production il est nécessaire d'avoir plusieurs serveurs Beacon, afin que les pairs puissent se découvrir même si un des serveurs tombe en panne. 

Pour simplifier le déploiement de ces serveurs, un seul fichier **cert_der.bin** peut être utilisé. Ce n'est pas obligatoire, mais cela simplifie la gestion des certificats.

En supposant que vous ayez déployé trois serveurs (firstbeacon.com, secondbeacon.com, thirdbeacon.com) avec le même fichier **cert_der.bin**, Discret pourra être [configuré](@/learn/configuration/configuration.fr.md) de la façon suivante (en supposant que le fichier de configuration est enregistré au format TOML):

```toml 
#Ipv4
[[beacons]]
hostname = "firstbeacon.com:4264"
cert_hash = "weOsoMPwj976xqxRvLElsbb-gijWWn0netOtgPflZnk"

#Ipv6
[[beacons]]
hostname = "ipv6.firstbeacon.com:4266"
cert_hash = "weOsoMPwj976xqxRvLElsbb-gijWWn0netOtgPflZnk"

#Ipv4
[[beacons]]
hostname = "secondbeacon.com:4264"
cert_hash = "weOsoMPwj976xqxRvLElsbb-gijWWn0netOtgPflZnk"

#Ipv6
[[beacons]]
hostname = "ipv6.secondbeacon.com:4266"
cert_hash = "weOsoMPwj976xqxRvLElsbb-gijWWn0netOtgPflZnk"


#Ipv4
[[beacons]]
hostname = "thirdbeacon.com:4264"
cert_hash = "weOsoMPwj976xqxRvLElsbb-gijWWn0netOtgPflZnk"

#Ipv6
[[beacons]]
hostname = "ipv6.thirdbeacon.com:4266"
cert_hash = "weOsoMPwj976xqxRvLElsbb-gijWWn0netOtgPflZnk"
```

Le **cert_hash** étant la valeur récupérée dans le fichier **certificate_hash.txt**
Vous noterez que chaque serveur est déclaré deux fois: 
- une fois pour le port d'écoute IPV4 
- une fois pour le port d'écoute IPV6

# Configuration de Beacon.conf.toml 

le fichier ne contient que trois paramètres:
- **ipv4_port**: le port d'écoute IPV4
- **ipv6_port**: le port d'écoute IPV6
- **num_buffers**: le nombre de buffer partagé par les connections. Augmenter ce nombre peut améliorer les performances si votre serveur est utilisé par de nombreuses personnes, mais ce au prix d'une augmentation de la consommation mémoire. Chaque buffer consommant 4kb de mémoire.

les changements dans ce fichier ne seront pris en compte qu'après le redémarrage du service.  


