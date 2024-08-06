+++
title = "Flutter chat with multiple users"
description = "Extends the chat example to support multiple users"
weight = 2
+++

This document presents an advanced chat example that allows multi-user chat. It describes the *example_advanced_chat.dart* available in the **./lib/** folder. 

It assumes that you have followed the [Minimalist Flutter Chat ](@/tutorial/flutter/flutter_chat.md) tutorial, and will only describes function that uses new parts of the *Discret* API.

As a starting point:
- replace the ./lib/main.dart by the new example file ./lib/example_advanced_chat.dart. 
- compile the application. This can take a few minutes to complete.

```
rm ./lib/main.dart
cp ./lib/example_advanced_chat.dart ./lib/main.dart
flutter build linux
```

Once compiled, do to folder do to the folder indicated at the end of the compilation. On a Linux device, the folder is the following:
```
cd build/linux/x64/release/bundle/
```

- launch the application a first time and create a new account. Compared to the previous example, you will notice that there is a new field named **displayed name**. It is the name that other peers will see. 
- keep the first instance open.
- launch the application a second time, and create another account.
- On the first instance, click on the **Invite** button, copy the invite string using the **Copy to Clipboard** and then click the **Ok** button.
- On the second instance, click on the **Accept Invite** button, paste the invitation in the field and click on the **Ok**
- on both instances, you should see the name of the other peer appearing in the left menu.
- you should now be able to discuss between the two accounts.

If you copy the program on another local device and re-create one of the account, you should be able to discuss between the two devices.

With three different accounts, the screen should look like that:
{{ image(path="/advanced_chat.png") }}

# Preparing the API

The Api initialization does not change much compared to the previous example.

The folder to store data is changed to support Android devices (and probably IOS).


```dart,linenos, linenostart=18
//...
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  final path = await _localPath;
  String dataPath = '$path/.discret_data';
```
---

We also set the log level to **info**. Log levels can be changed anytime with the values:
- **error**
- **warn**
- **info**
- **debug**
- **trace**

```dart,linenos, linenostart=27
  await Discret().setLogLevel("info");
```
# Architecture

To manage several discussion groups, the **ChatState** provides a list of **ChatRoom** and all the functions required to
- connect
- manage invite
- manage the discussion group list

```dart
class ChatState extends ChangeNotifier {
  final List<ChatRoom> chatRooms = [];
//..
}
```

The **ChatRoom** the fields and function required to send and receive messages.

```dart
class ChatRoom {
  final List<ChatEntry> chat = [];
  String roomId;
  String name;
  int mdate;
}
```

# Initialization
The **ChatState** initialization is more complex that the previous one.

During the ChatSate creation, a log listener is created. Those logs are primarily used to send error message that are not directly related to an API call.

The code only push the *logs* in the Flutter log Flutter log for debugging purposes.

```dart,linenos, linenostart=50
  final StreamSubscription<LogMsg> loglistener =
      Discret().logStream().listen((event) async {
    var message = event.data;
    var level = event.level;
    logger.log('$level: $message');
  });
```

--

Line 68 retrieve the user's **verifyingKey**. It represents the public identity of this user. 

Data created by this user are signed with the associated  signature key and other peers will verify the signature during data synchronization.

```dart
    msg = await Discret().verifyingKey();
    myVerifyingKey = msg.data;
```


---
The event stream is also more complex and manage more events.

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
The **EventType.dataChanged** event have take into account the different discussion groups and will only refresh the *Rooms* that have new data. 

A new event is managed: **EventType.roomModified**. This event is triggered by the invitation functions when creating new invites and accepting invites. It detects that *Rooms* have been created or modified.

The events **EventType.peerConnected** and **EventType.peerDisconnected** are triggered when a peer connect or disconnect. It is not directly used by the code but put in the flutter log for debugging purposes.



# Creating an Account

The login function does not change much, but the account creation is more complex because it has to manage the new **Display Name** field.

This field is used to associate a name to your public identity (your verifying_key). Your identity and name will be sent to other peers when 
accepting invitations.

This association is done in the **sys.Peer** that contains all known peers, including you.

```dart,linenos, linenostart=128
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

The first query line 133 retrieve the unique identifier **id** of the tuple that stores your identity.

Every tuple in the *Discret* database have a unique **id**  generated during the insertion.

```dart
String query = """query{
  sys.Peer(verifying_key=\$verifyingKey){
    id
  }
}""";
```
---

The second query line 150 updates the **name** field with the **Display Name** value.
Every **mutation** query that provides an **id** is an update query. If the **id** is not found, an error will be returned.

```dart
String mutate = """mutate {
  sys.Peer{
    id:\$id
    name:\$name
  }
}""";
```


# Creating an invite

Every new invites needs to create a new [Room](@/learn/access_rights/room.md) to limit access to to discussion group to you and the invited peer.


```dart,linenos, linenostart=273
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

The query ligne 274 creates a new room for the invited peer.

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
When creating the *Room*, the invited peer identity (its **verifying_key**) is not yet known, so you just insert your identity.

You can notice that your identity is used twice:
- once in the **admin** field, giving you administrator rights for this *Room*
- once in the **user** field of the autorisation. It is not strictly necessary because administrator have every rights, but it simplifies the code in this example. In the list of available group, we will only display groups that have two **user**, indicating that the invite has been accepted.
  
The **rights** field, indicate that only tuples of the **chat.Message** entity are allowed in this room. Trying to insert other things will return an error. 

---
The API to create an invite requires:
- The *Room* **id** de la *Room*,
- The **id** of an autorisation that belongs to that *Room*.

Those identifiers are retrieved by parsing the result of the query  (lines 293 to 307) that contains every ids generated during insertion.

Once the invite accepted, the new peer identity will be inserted in the **user** field of the provided autorisation.

---

Calling the API will retur a large invitation String that you will need to *manually* transmit to the invited peer 

```dart
 ResultMsg invite = await Discret().invite(roomId, authId);
  //..
 return invite.data;
```


# Accepting an invitation

Accepting an invitation is just an API call with the invitation string. 

An invitation can only be used once.

```dart,linenos, linenostart=320
  Future<void> acceptInvite(String invite) async {
    ResultMsg msg = await Discret().acceptInvite(invite);
    if (!msg.successful) {
      logger.log(msg.error);
    }
  }
```