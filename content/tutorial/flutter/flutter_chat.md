+++
title = "Minimalist Flutter Chat"
description = "Start learning the Discret Flutter API by creating a simple chat application"
weight = 1
+++

This document considers that you have followed the previous [Installing Components ](@/tutorial/flutter/instal.md) document. 

At the end of the previous document, you should have copied *example_simple_chat.dart* in the main.dart file and run the application.

```
cp ./lib/example_simple_chat.dart ./lib/main.dart
flutter run
```

After creating an account, you should see the following screen:

{{ image(path="/simple_chat.png") }}


If you copy this program in different folders or different devices on your local network, you should be able to discuss between the different instances. 

This document does not explain how to create the Flutter screens but focus on the code that is specific to the *Discret* API.
For simplicity reasons error handling is very basic. A real application would need to have better error handling mechanisms.


# Preparing the API
The Discret API requires three parameters:


- **a data model**: defines the kind of data that can be used in *Discret*,
- **an application unique identifier**: once the application in production, this identifier cannot be changed anymore. It is used to derive some secrets that will be used by *Discret*. If this identifier is changed, the secrets will change and users will not be able to connect anymore.
-  **a data folder**: where data is stored.


The beginning of the file initialize *Discret*:
```dart,linenos, linenostart=8
//...
import './messages/discret.pb.dart';
import 'discret.dart';
// the application unique identifier
// ignore: constant_identifier_names
const String APPLICATION_KEY = "github.com/discretlib/flutter_example_simple_chat";
void main() async {
  // does not work on smartphone, 
  // example_advanced_chat.dart provides a version compatible with smartphone
  String dataPath = '.discret_data';
  String datamodel = """chat {
    Message {
      content: String
    }
  }""";
  await Discret().initialize(APPLICATION_KEY, dataPath, datamodel);
  
  runApp(
    const ChatApp(),
  );
}

//
// This class contains every API calls.
//
class ChatState extends ChangeNotifier {
//...
```

Lines 9 and 10 import the two packages of the API.


Line 13 defines the application unique identifier:
```dart,linenos,linenostart=12
//...
const String APPLICATION_KEY = "github.com/discretlib/flutter_example_simple_chat";
//...
```
---

Lines 18 to 22 define the data model used by this application. It creates a namespace **chat** that contains an Entity names **Message**, which contains an unique field named **content** of the String type. 


You can learn more about data models in the documentation [Schema and Entities](@/learn/datamodel/schema.md).

```dart,linenos,linenostart=17
//...
  String datamodel = """chat {
    Message{
      content:String
    }
  }""";
//...
```

---

One the API initialized, The application will start and instantiate the  **ChatState** class that contains every functions that calls the *Discret* API

# The ResultMsg class

You will see in the rest of the document that most of the API calls returns an **ResultMsg** object. It contains the following fields.

```dart
class ResultMsg{
    int id,
    bool successful,
    String error,
    String data,
}
```

- **id** is a technical field not useful to the user,
- **successful** indicate whether the API call was a success or not,
- **error** contains the error message, if any;
- **data** contains the result of the API call.



# Creating an Account and connecting

Creating an account and connecting to the application uses the same API call.

```dart,linenos,linenostart=47
  Future<ResultMsg> login(String userName, String password, bool create) async {
    ResultMsg msg = await Discret().login(userName, password, create);
    if (msg.successful) {
      await initialise();
    }
    return msg;
  }
```

The difference between an account creation and a connection is the **create**  value:
- **false**: connect if the account exists and return an error otherwise,
- **true**: create a new account if it does not exists and return an error if it already exists.

If the connection is successful, the *initialise()* function is called. The *Discret* library only starts  after the authentication, so we have to wait for a successful to finalise **ChatState** initialisation.

# *ChatState* Initialization 

The initialise function is the following:

```dart,linenos,linenostart=58
Future<void> initialise() async {
  ResultMsg msg = await Discret().privateRoom();
  privateRoom = msg.data;
  
  eventlistener = Discret().eventStream().listen((event) async {
      switch (event.type) {
      case EventType.dataChanged:
          await refreshChat();
  
      default:
      }
  }, onDone: () {}, onError: (error) {});

  refreshChat();
}
```

Line 60 retrieves the user's private *Room*. *Rooms* are the access rights system used to synchronized data with other peers. Data inserted in a *Room* is synchronized only with peers that have access rights to it. 

In this example we will only use the user's private *Room*, meaning that only him will able to access the data. If this user's is connected on several devices, data will be synchronized between  devices.

You can learn more about access right in the [Room](@/learn/access_rights/room.md) section of the documentation.

---

Starting ligne 62, we will listen to the *Discret* event stream. We are only interested in the  **EventType.dataChanged** event that is triggered when Discret detect that data have been modified.

In this example, this event is triggered when:
- you are sending a message from this instance
- when data from other instances are synchronized with this one

When this event is detected, we call the **refreshChat()** function that will read the new data.

You can learn more about the different types of events in the [System Events ](@/learn/events.md) section of the documentation.

---
Once the initialisation done, a first call to **refreshChat()** is performed to read messages that could have been inserted during a previous session.

# Read Messages: the refreshChat() function

The function used to read messages is the following:

```dart,linenos,linenostart=77
Future<void> refreshChat() async {
  String query = """query {
          res: chat.Message(
              order_by(mdate asc, id asc), 
              after(\$mdate, \$id),
              room_id = \$room_id
          ) {
                  id
                  mdate
                  content
          }
          }""";
  ResultMsg res = await Discret().query(
      query, {"mdate": lastEntryDate, "id": lastId, "room_id": privateRoom});
  
  if (!res.successful) {
      print(res.error);
  } else {
      final json = jsonDecode(res.data) as Map<String, dynamic>;
      final msgArray = json["res"] as List<dynamic>;
  
      for (var msgObject in msgArray) {
      var message = msgObject as Map<String, dynamic>;
      var entry =
          ChatEntry(message["id"], message["mdate"], message["content"]);
      lastEntryDate = entry.date;
      lastId = entry.id;
  
      chat.add(entry);
      }
      notifyListeners();
  }
}
```

The function uses a  **query** to read data from the discret database. You can learn more about reading database data in the [Query](@/learn/datamodel/query.md) section of the documentation.

You can notice that the query defines three parameters: 
- **\$mdate** 
- **\$id**
- **\$room_id**

that are passed during the API call:
```dart
ResultMsg res = await Discret().query(
    query, {
        "mdate": lastEntryDate,
        "id": lastId, 
        "room_id": privateRoom
    }
);
```

# Sending a message

The function used to write messages is the following:

```dart,linenos,linenostart=113
Future<void> sendMessage() async {
  if (chatController.text != "") {
      String query = """mutate {
        res : chat.Message{
            room_id: \$room_id
            content: \$content
        }
      }""";
  
      ResultMsg res = await Discret().mutate(
          query, {"room_id": privateRoom, "content": chatController.text});
  
      if (!res.successful) {
      print(res.error);
      } else {
      chatController.clear();
      }
  }
}
```
It used a **mutation** query to insert a new tuple of the **chat.Message** entity defined in the data model.

You can notice that the **room_id**  has not been manually defined in the data model. It is a system field available for every entities. **\$room_id** and **\$message** are query parameters that are passed by the **Some(params)** object.


**\$room_id** et **\$message** sont are two query parameter that are passed during the API call: 
```dart
ResultMsg res = await Discret().mutate(
  query, {
    "room_id": privateRoom, 
    "content": chatController.text
  }
);
```
You can lean more about data insertion and modification in the [Mutations and Deletions](@/learn/datamodel/mutation.md) section of the  documentation.


