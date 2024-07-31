+++
title = "Chat Multi Utilisateurs"
description = "Etendez l'exemple de base pour obtenir un chat gérant plusieurs utilisateurs"
weight = 2
+++

Ce document décrit l'exemple avancé de chat nommé *example_advanced_chat.dart* disponible dans le répertoire **./lib/**, 
 et considère que vous avez déjà suivi le tutoriel [Chat Flutter Minimaliste](@/tutorial/flutter/flutter_chat.fr.md). Seules les fonctions introduisant de nouvelles parties de l'API seront décrites.

Pour commencer:
- remplacez ./lib/main.dart par le nouveau fichier d'exemple ./lib/example_advanced_chat.dart. 
- compilez ensuite l'application. cela peut prendre plusieurs minutes

```
rm ./lib/main.dart
cp ./lib/example_advanced_chat.dart ./lib/main.dart
flutter build linux
```

Ensuite, allez dans le répertoire indiqué à la fin de la compilation, pour Linux:
```
cd build/linux/x64/release/bundle/
```


- Lancez un première fois l'application *discret_flutter* et créez un nouveau compte. Par rapport au précédent exemple, vous noterez qu'un nouveau champ nommé **displayed name** est apparu. Il s'agit du nom que les autres Pairs verront.
- Gardez la première application ouverte, lancez l'application une seconde fois, et créez un second compte.
- Sur la première fenêtres, cliquez sur le bouton **Invite**, copiez l'invitation en cliquant sur **Copy to Clipboard** et fermez la fenêtre en cliquant sur **Ok**
- Sur la seconde fenêtre, cliquez sur le bouton **Accept Invite**, collez l'invitation dans le champs prévu à cet effet et appuyez sur **Ok**.
- Vous devriez voir les noms des comptes apparaître dans les deux fenêtres, indiquant que l'invitation a été acceptée.
- Vous devriez pouvoir discuter entre les deux comptes.

Si vous copiez le programme sur une autre machine du réseau et que vous re-créez un des comptes, vous devriez pouvoir discuter entre vos deux machines, avec deux comptes différents. 

Pour trois utilisateurs, vous devriez voir l'écran:
{{ image(path="/advanced_chat.png") }}

# Préparation de l'API

L'initialisation de l'API ne change pas beaucoup par rapport à l'exemple précédent.

Seul le répertoire où stocker des données est modifié pour être compatible avec les *smartphone* et *tablet*.

```dart,linenos, linenostart=18
//...
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  final path = await _localPath;
  String dataPath = '$path/.discret_data';
  ```

# Architecture
Pour gérer les différents groupes de discussion, la classe  **ChatState** contient une liste de **ChatRoom** ainsi que tous les champs et fonctions pour gérer:
- la connections
- les invitations
- la gestion de la liste des groupes de discussion.

```dart
class ChatState extends ChangeNotifier {
  final List<ChatRoom> chatRooms = [];
//..
}
```

La classe **ChatRoom** contient tous les champs et fonctions pour:
 - envoyer des message 
 - recevoir les messages.

```dart
class ChatRoom {
  final List<ChatEntry> chat = [];
  String roomId;
  String name;
  int mdate;
}
```

# Initialisation
L'initialisation du **ChatState** est plus complexe que la précédente.

La ligne 60 récupère la **verifyingKey** de l'utilisateur connecté. Cela représente l'identité publique de l'utilisateur.

Toutes les données insérés sont signées avec la clé de signature associée, et les autres utilisateurs vérifieront cette signature lors de la synchronisation des données.

```dart,linenos, linenostart=59
//...
    msg = await Discret().verifyingKey();
    myVerifyingKey = msg.data;
//..
```

---
La gestion du flux d'évènements envoyé par *Discret* est plus complète.

```dart
eventlistener = Discret().eventStream().listen((event) async {
    switch (event.type) {
      case EventType.dataChanged:
        {
          DataModification modif = event.data as DataModification;
          modif.rooms?.forEach((key, value) {
            if (chatMap.containsKey(key)) {
              ChatRoom chatRoom = chatMap[key]!;
              chatRoom.refresh();
            }
          });
          chatRooms.sort((a, b) => b.mdate.compareTo(a.mdate));
          notifyListeners();
        }
      case EventType.roomModified:
        {
          String roomId = event.data as String;
          await loadRoom(roomId);
          notifyListeners();
        }
      case EventType.peerConnected:
        {
          PeerConnection peer = event.data as PeerConnection;
          var key = peer.verifyingKey;
          logger.log('Peer connected: $key');
        }
      case EventType.peerDisconnected:
        {
          PeerConnection peer = event.data as PeerConnection;
          var key = peer.verifyingKey;
          logger.log('Peer disconnected: $key');
        }
      default:
    }
  }, onDone: () {}, onError: (error) {});
```
L'évènement **EventType.dataChanged** doit maintenant prendre en compte les différent groupe de discussion et ne rafraîchira que les **Rooms** qui ont de nouveaux messages.

Un nouvel évènement est traité **EventType.roomModified**. Cet évènement sera envoyé pendant le traitements des invitations et permet de détecter qu'un nouveau groupe de discussion est disponible. 

Les évènements **EventType.peerConnected**, **EventType.peerDisconnected** permettent de détecter qu'un Pair vient de se connecter ou de se déconnecter. Ils ne sont pas utilisés dans le code et ne font qu'écrire dans le log.

---

L'initialisation définit ensuite la gestion du flux de *logs* envoyé par *Discret*.
Ces *logs* sont principalement utilisés pour renvoyer des erreurs techniques qui ne sont pas particulièrement liées à un appel de l'API.

Le code suivant ne fait que renvoyer ces *logs* dans le log Flutter de deboggage.

```dart
loglistener = Discret().logStream().listen((event) async {
  var date = DateTime.fromMillisecondsSinceEpoch(event.date);
  var message = event.message;
  switch (event.type) {
    case LogType.info:
      message = 'INFO: $message';

    case LogType.error:
      var source = event.source;
      message = 'ERROR: source: $source message: $message';
  }
  logger.log(message, time: date);
});
```

# Créer un Compte

La fonction de login n'a pas changée mais la fonction de création de compte est plus complexe car elle doit désormais gérer le champ **Display Name**.

Ce champ est utilisé pour associer un nom à votre identité publique (votre verifying_key). Votre identité ainsi que ce nom seront envoyés aux Pairs lors du traitement des invitations.

Cette association se fait dans l'entité **sys.Peer** qui contient tous les Pairs que vous connaissez, vous y compris.

```dart,linenos, linenostart=135
 Future<ResultMsg> createAccount(
      String userName, String password, String displayName) async {
    ResultMsg msg = await Discret().login(userName, password, true);
    if (msg.successful) {
      await initialise();
      String query = """query{
        sys.Peer(verifying_key=\$verifyingKey){
          id
        }
      }""";

      ResultMsg res =
          await Discret().query(query, {"verifyingKey": myVerifyingKey});
      if (!res.successful) {
        logger.log(res.error);
        return res;
      }

      final json = jsonDecode(res.data) as Map<String, dynamic>;
      final msgArray = json["sys.Peer"] as List<dynamic>;
      var result = msgArray[0] as Map<String, dynamic>;
      var id = result["id"];
      String mutate = """mutate {
        sys.Peer{
          id:\$id
          name:\$name
        }
      }""";
      res = await Discret().mutate(mutate, {"id": id, "name": displayName});
      if (!res.successful) {
        logger.log(res.error);
        return res;
      }
      return res;
    }
    return msg;
  }
```
---

La première requête ligne 140 va récupérer l'identifiant **id** du tuple correspondant à votre identité.

Chaque tuple en base de données possède un identifiant unique généré lors de l'insertion.

```dart
String query = """query{
  sys.Peer(verifying_key=\$verifyingKey){
    id
  }
}""";
```
---

La seconde requête ligne 157 va mettre à jour le champ **name** avec la valeur de **Display Name**.

Toute requête de type **mutate** qui fournit un **id** est une requête de mise à jours. Si l'identifiant n'existe pas en base de données, une erreur est retournée.

```dart
String mutate = """mutate {
  sys.Peer{
    id:\$id
    name:\$name
  }
}""";
```


# Créer une invitation

Chaque demande d'invitation va créer une nouvelle [Room](@/learn/access_rights/room.fr.md) qui va permettre de limiter les droits d'accès au groupe de discussion.


```dart,linenos, linenostart=280
  Future<String> createInvite() async {
    String query = """mutate {
      sys.Room{
        admin: [{
          verif_key:\$verifyingKey
        }]
        authorisations:[{
          name:"chat"
          rights:[{
            entity:"chat.Message"
            mutate_self:true
            mutate_all:false
          }]
          users:[{
              verif_key:\$verifyingKey
          }]
        }]
      }
    }""";

    ResultMsg res =
        await Discret().mutate(query, {"verifyingKey": myVerifyingKey});

    if (!res.successful) {
      logger.log(res.error);
      return "";
    }

    final json = jsonDecode(res.data) as Map<String, dynamic>;
    final msgArray = json["sys.Room"] as Map<String, dynamic>;
    String roomId = msgArray["id"];

    final auths = msgArray["authorisations"] as List<dynamic>;
    final authorisation = auths[0] as Map<String, dynamic>;
    final authId = authorisation["id"];

    ResultMsg invite = await Discret().invite(roomId, authId);
    if (!res.successful) {
      logger.log(res.error);
      return "";
    }
    return invite.data;
  }
```

La requête ligne 281 va créer une nouvelle Room pour accueillir le nouvel invité.

```dart
String query = """mutate {
  sys.Room{
    admin: [{
      verif_key:\$verifyingKey
    }]
    authorisations:[{
      name:"chat"
      rights:[{
        entity:"chat.Message"
        mutate_self:true
        mutate_all:false
      }]
      users:[{
          verif_key:\$verifyingKey
      }]
    }]
  }
}""";
```
Lors de la création de la *Room*, l'identité (**verifying_key**) du pair invité est encore inconnue, vous n'insérez donc que votre propre identité.

Vous pourrez noter que votre identité est utilisée deux fois:
- une fois dans le champ **admin**, indiquant que vous être administrateur de cette *Room*,
- une fois dans le champ **user** de l'autorisation. Ce n'est pas vraiment nécessaire car les administrateurs ont tous les droits, mais cela simplifie le code de cet exemple. Dans la liste des groupe de chat nous n'afficheront que les **Rooms** possédant deux entrées dans le champs **user**, car cela indique que l'invitation a été acceptée.

Le champ **rights** de l'autorisation indique que seul les tuples de type **chat.Message** sont acceptés. Tenter d'insérer d'autres données renverra une erreur.


---

L'API de creation d'invitation requiers:
- l'**id** de la *Room*,
- l'**id** d'une autorisation interne à cette *Room*.

Ces identifiants sont récupérés en lisant le résultat de la requête de création (lignes 308 à 314), qui contient les identifiants générés lors de la création.

Lorsque l'invitation sera acceptée, l'identité du Pair sera rajoutée automatiquement à l'autorisation fournie, vous pourrez commencer à communiquer en insérant des messages dans cette *Room*.

---

L'appel de la fonction de l'API va retourner une grande chaîne de caractères qui devra être transmise au pair que vous voulez inviter par des moyens externe à l'application.
```dart
 ResultMsg invite = await Discret().invite(roomId, authId);
  //..
 return invite.data;
```


# Accepter une invitation

Accepter une invitation se résume à appeler la fonction de l'API avec l'invitation reçue. L'invitation va permettre aux deux pairs de se retrouver sur le réseau et se connecter. 

Une invitation ne peut être utilisée qu'une seule fois.

```dart,linenos, linenostart=18
  Future<void> acceptInvite(String invite) async {
    ResultMsg msg = await Discret().acceptInvite(invite);
    if (!msg.successful) {
      logger.log(msg.error);
    }
  }
```