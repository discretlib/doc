+++
title = "Chat Flutter Minimaliste"
description = "Commencer à apprendre L'API Flutter en construisant une application de chat minimaliste"
weight = 1
+++

Ce document considère que vous avez suivi le document [Installer les composants](@/tutorial/flutter/instal.fr.md).

A la fin du précedent document vous devez avoir copié le fichier example_simple_chat.dart dans le fichier main.dart et lancé l'application.

```
cp ./lib/example_simple_chat.dart ./lib/main.dart
flutter run
```

Après avoir créé un compte, vous devriez obtenir l'écran suivant:

{{ image(path="/simple_chat.png") }}


Si vous copiez le programme dans plusieurs répertoires ou machines en réseau local, et que vous créez le même compte à chaque fois, vous devriez être capable de converser entre les différents clients. Cet example est peu utile, mais permettra de vous familiariser avec l'API *Discret*.

Ce document ne décrit pas comment créer les écrans *Flutter* mais va se concentrer sur le code spécifique à l'API *Discret*. 

Pour des raisons de simplicité, la gestion des erreurs est très basique, une application réelle devra mieux les gérer.


# Preparation de l'API
l'API Discret nécessite trois paramètres pour être initalisée:

- **Un identifiant unique pour l'application**: une fois définit, cet identifiant ne devra jamais changer une fois l'application mise en production. Cet identifiant est utilisé pour dériver des secrets utilisés par *Discret*. Si cet identifiant change, les utisateurs ne pourront plus se connecter.
- **Un modèle de données**: définit les types de données qui peuvent être utilisés dans *Discret*.
- **Un repertoire**: où stocker des données.

Le debut du fichier initialise *Discret*
```dart,linenos, linenostart=8
//...
import './messages/discret.pb.dart';
import 'discret.dart';
// L'identifiant unique de l'application
// ignore: constant_identifier_names
const String APPLICATION_KEY = "github.com/discretlib/flutter_example_simple_chat";
void main() async {
  //ne fonctionne pas sur smartphone, 
  // example_advanced_chat.dart fournit un version compatible avec les smartphone
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
// Cette classe contient tous les appels à l'API.
//
class ChatState extends ChangeNotifier {
//...
```

les lignes 9 et 10 importent les deux packages requis par l'API.


La ligne 13 définit l'identifiant unique de l'application:
```dart,linenos,linenostart=12
//...
const String APPLICATION_KEY = "github.com/discretlib/flutter_example_simple_chat";
//...
```
---

Les lignes 18 à 22 définissent le modèle de données qui pourra être utilisé dans l'application. 

Ce modèle définit un espace de nom nommé **chat** qui contient l'entité **Message**, qui elle même contient un unique champ nommé **content** qui peut contenir une chaine de caractères. 


```dart,linenos,linenostart=17
//...
  String datamodel = """chat {
    Message{
      content:String
    }
  }""";
//...
```
Vous pourrez en apprendre plus dans la section [Schémas et Entités](@/learn/datamodel/schema.fr.md) de la documentation.

Une fois l'API initialisée, l'application va démarrer et instancier la classe **ChatState** qui contient toutes les fonctions qui vont appeller l'API. 

# L'objet ResultMsg

Vous verrez dans la suite du document que la plupart des fonctions de l'API retournent l'object **ResultMsg** qui contient les champs suivants.

```dart
class ResultMsg{
    int id,
    bool successful,
    String error,
    String data,
}
```

- **id** est un champ technique sans utilité particulière pour l'utilisateur,
- **successful** indique si la fonction s'est effectuée avec succès,
- **error** contient le message d'erreur eventuel,
- **data** contient le résultat de l'appel.



# Créer un compte et se connecter

La création d'un compte et la connection se font avec une unique fonction de l'API.

```dart,linenos,linenostart=47
  Future<ResultMsg> login(String userName, String password, bool create) async {
    ResultMsg msg = await Discret().login(userName, password, create);
    if (msg.successful) {
      await initialise();
    }
    return msg;
  }
```
La différence entre une création de compte et une connection réside dans la valeur de **create**:
- **false**: la fonction se connectera si le compte existe, et retournera une erreur si il n'existe pas
- **true**: la fonction va créer le compte et se connecter si il n'existe pas, et retournera une erreur si le compte existe déjà.

Si la connection est un succès, la fonction *initialise();* est appelée. La librairie **Discret** ne démarre qu'après l'authentification, donc le **ChatState**  ne peut pas être initialisé avant.

# Initialisation du *ChatState*  

La fonction d'initialisation est la suivante:

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

La ligne 60 récupère la *Room* privée de l'utilisateur. Le concept de Room définit le système d'autorisations utilisé par *Discret* pour synchroniser les données avec ses Pairs. Les données insérées dans une *Room* ne seront synchronisées qu'avec les Pairs ayant accès à cette *Room*. 

Nous allons insérer les messages dans la *Room* Privée de l'utilisateur, donc lui seul sera capable d'y accéder. Si l'utilisateur se connecte sur plusieurs machines, les données seront synchronisées entre les machines. 

Vous pourrez en apprendre plus dans la section [Room](@/learn/access_rights/room.fr.md) de la documentation.

---
A partir de la ligne 62, nous allons écouter le flux d'évènements envoyé par *Discret*. Nous ne nous intéressons qu'à l'évènement **EventType.dataChanged** qui est envoyé quand *Discret* détecte que des données ont été modifiées. 

Dans cet exemple, cet évènement est déclenché:
- quand vous insérez un message depuis cette instance de l'application
- quand des données d'autre instance de l'application sont synchronisées avec cette instance

Quand cet évènement est détecté, la fonction **refreshChat()** est appelée pout mettre à jours la liste des messages.

 Vous pourrez en apprendre plus sur les différents évènements dans la section [Évènements Système](@/learn/events.fr.md) de la documentation.

---
Une fois l'initialisation terminée la fonction **refreshChat()** est appelée un première fois pour récupérer les messages qui ont pu être écrits lors d'une session précédente.


# Récupérer les messages: la fonction refreshChat()


La fonction utilisée pour récupérer les messages est la suivante:

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

La fonction envoie une requête de type **query** qui permet de récupérer des données depuis la base de donnée de *Discret*. Vous pourrez en apprendre plus sur les requêtes de sélection de données dans la section [Requête](@/learn/datamodel/query.fr.md) de la documentation.

Vous pourrez noter que cette requête définit trois paramètres: 
- **\$mdate** 
- **\$id**
- **\$room_id**

qui sont passés lors de l'appel à l'API:

```dart
ResultMsg res = await Discret().query(
    query, {
        "mdate": lastEntryDate,
        "id": lastId, 
        "room_id": privateRoom
    }
);
```

# Envoyer un message

La fonction utilisée pour envoyer un message est la suivante:
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
Nous utilisons une requête de type **mutation** pour insérer un tuple de l'entité **chat.Message** définie dans le modèle de données. 

Vous noterez que **room_id** n'a pas été défini dans le modèle de données. C'est un champ système disponible pour toutes les entités.  

**\$room_id** et **\$message** sont des paramètres de la requête qui sont passés lors de l'appel à l'API: 
```dart
ResultMsg res = await Discret().mutate(
  query, {
    "room_id": privateRoom, 
    "content": chatController.text
  }
);
```
Vous pourrez en apprendre plus sur l'insertion et la modification de données dans la section [Mutations et Suppression](@/learn/datamodel/mutation.fr.md) de la documentation.

