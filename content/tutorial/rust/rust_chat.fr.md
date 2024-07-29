+++
title = "Chat Minimaliste"
description = "Créez un application de chat minimale utilisant Rust"
weight = 1
+++

Nous allons créer un application de chat vraiment minimaliste qui ne permettra de discuter qu'avec vous même. Bien que peut utile, cela permet d'introduire les différents mécanismes utilisés par *Discret*.

# Préparation
Pour commencer, nous allons créer un nouveau projet rust nommé  *rust_simple_chat*:
```
cargo new rust_simple_chat
cd rust_simple_chat
```

Editez le fichier **Cargo.toml** pour rajouter les dépendances suivantes:
```toml
[dependencies]
    discret = "0.6.0"
    tokio = { version = "1.38.0", features = ["full"] }
    serde = { version = "1.0.203", features = ["derive"] }
    serde_json = "1.0.117"
```

# Initialiser *Discret*
Discret nécessite un certain nombre de paramètres pour être lancé:
- **Un modèle de données**: définit les types de données qui peuvent être utilisés dans *Discret*.
- **Un identifiant unique pour l'application**: une fois définit, cet identifiant ne devra jamais changer une fois l'application mise en production. Cet identifiant est utilisé pour dériver des secrets utilisés par *Discret*. Si cet identifiant change, les utisateurs ne pourront plus se connecter.
-  **Un secret maître**: ce secret sera utilisé avec l'identifiant unique pour dériver les secrets de l'utilisateur. Nous utiliserons dans cet exemple un secret dérivé d'un nom d'utilisateur et d'un mot de passe.
-  **Un repertoire**: où stocker des données.
-  **Une configuration**: nous utiliserons la configuration par défaut.


Remplacer le fichier **src/main.rs** par défaut par le suivant 
```rust,linenos
use std::{io, path::PathBuf};

use discret::{
  derive_pass_phrase, zero_uid, Configuration, 
  Discret, Parameters, ParametersAdd, ResultParser,
};
use serde::Deserialize;

//identifiant unique de l'application
const APPLICATION_KEY: &str = "github.com/discretlib/rust_example_simple_chat";

#[tokio::main]
async fn main() {
  //definition du modèle de données
  let model = "chat {
    Message{
      content:String
    }
  }";

  let path: PathBuf = "test_data".into(); //où les donnée sont stockées

  //utilisé pour dériver les tous les secrets nécessaires au fonctionnement de Discret
  let key_material: [u8; 32] = derive_pass_phrase("login", "password");

  //démarre Discret 
  let app: Discret = Discret::new(
    model,
    APPLICATION_KEY,
    &key_material,
    path,
    Configuration::default(),
  ).await.unwrap();
}
```

La ligne 10 definit un nom unique de l'application:
```rust,linenos,linenostart=9
//...
const APPLICATION_KEY: &str = "github.com/discretlib/rust_example_simple_chat";
//...
```

Les lignes 15 à 19 définissent le modèle de données qui pourra être utilisé dans l'application. Ce modèle définit un espace de nom nommé **chat** qui contient l'entité **Message**, qui elle même contient un unique champ nommé **content** qui peut contenir une chaine de caractères. 

Vous pourrez en apprendre plus dans la section [Schémas et Entités](@/learn/datamodel/schema.fr.md) de la documentation.
```rust,linenos,linenostart=14
//...
  let model = "chat {
    Message{
      content:String
    }
  }";
//...
```

La Ligne 24 créee le secret maitre. Ce secret est utilisé par Discret pour créer, en autre:
- La clé de chiffrement de la base de données. Toutes les donnes sont cryptées avant d'être stockées
- Un couple clé publique/clé privé qui sera utilisé pour signer toutes les données insérées et modfiées par cet utilisateurs. Toutes les données échangées par les pairs sont signées et vérifiées. 

la fonction **derive_pass_phrase** utilise la fonction de dérivation Argon2id avec les paramètres recommandés par [owasp.org](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)

```rust,linenos,linenostart=23
//...
let key_material: [u8; 32] = derive_pass_phrase("login", "password");
//...
```

L'application ne fait rien encore, mais *Discret* est maintenant prêt à être utilisé.

# Insérer des messages
Nous allons maintenant pouvoir insérer des données. Le programme va lire les messages écrit dans la console et les insérer en base de données. 

Insérer les lignes suivantes apres la ligne 33 **).await.unwrap()**;

```rust,linenos,linenostart=34
  let private_room: String = app.private_room();
  let stdin = io::stdin();
  let mut line = String::new();
  println!("{}", "Write Something!");
  loop {
    stdin.read_line(&mut line).unwrap();
    if line.starts_with("/q") {
    break;
    }
    line.pop();
    let mut params = Parameters::new();
    params.add("message", line.clone()).unwrap();
    params.add("room_id", private_room.clone()).unwrap();
    app.mutate(
    "mutate {
        chat.Message {
            room_id:$room_id 
            content: $message 
        }
    }",
    Some(params),
    ).await.unwrap();

    line.clear();
  }
```

La ligne 34 récupère la *Room* privée de l'utiliteur. Le concept de Room définit le système d'autorisations utilisé par *Discret* pour synchroniser les données avec ses Pairs. Les données insérées dans une *Room* ne seront synchronisées qu'avec les Pairs ayant accès à cette *Room*. 

Nous allons utiliser la *Room* Privée de l'utilisateur, donc lui seul sera capable d'y accèder. Si l'utilisateur se connecte sur plusieurs machines, les données seront synchronisées entre les machines. 

Vous pourrez en apprendre plus dans la section [Room](@/learn/access_rights/room.fr.md) de la documentation.

Les lignes 47 à 57 définissent la requête utilisée pour insérer des messages:
```rust,linenos,linenostart=46
    //...
    app.mutate(
    "mutate {
        chat.Message {
            room_id: $room_id 
            content: $message 
        }
    }",
    Some(params),
    ).await.unwrap();
    //...
```

Nous utilisons une requête de type **mutation** pour insérer un tuple de l'entité **chat.Message** définie dans le modèle de données. 

Vous noterez que **room_id** n'a pas été defini dans le modèle de données. C'est un champ système disponible pour toutes les entités.  **\$room_id** et **\$message** sont des paramêtres de la requête qui sont passés par l'objet **Some(params)**.

Vous pourrez en apprendre plus sur l'insertion et la modification de données dans la section [Mutations et Suppression](@/learn/datamodel/mutation.fr.md) de la documentation.

Si vous lancez l'application, vous devrier pouvoir écrire des messages
```
cargo run
```

# Lire les données synchronisées

A ce stade, les données que vous insérez seront synchronisées entre chaque instance de l'application. Si vous copiez le programme dans deux répertoires ou appareils différents, les données seront synchronisées. Par contre vous ne verrez pas les messages des autres.

Pour récupérer les messages synchronisés, nous allons écouter les évènements envoyé par **Discret**, et lire les messages quand un evenements indiquera que de nouvelles données sont disponibles. Pour ce faire, insérez le code suivant entre la ligne 33 **).await.unwrap();** et la ligne 34: **let private_room: String = app.private_room();**
```rust ,linenos,linenostart=33
//cette structure permet de désérialiser les messages
#[derive(Deserialize)]
struct Chat {
  pub id: String,
  pub mdate: i64,
  pub content: String,
}

//écoute les évènements créés par Discret
let mut events = app.subscribe_for_events().await;
let event_app: Discret = app.clone();
tokio::spawn(async move {
  let mut last_date = 0;
  let mut last_id = zero_uid();

  let private_room: String = event_app.private_room();
  while let Ok(event) = events.recv().await {
    match event {
      //déclenché quand des données sont insérées ou modifées
      discret::Event::DataChanged(_) => {
        let mut param = Parameters::new();
        param.add("mdate", last_date).unwrap();
        param.add("id", last_id.clone()).unwrap();
        param.add("room_id", private_room.clone()).unwrap();

        //obtiens les dernières données dans le format JSON 
        let result: String = event_app.query(
          "query {
              res: chat.Message(
                  order_by(mdate asc, id asc), 
                  after($mdate, $id),
                  room_id = $room_id
              ) {
                      id
                      mdate
                      content
              }
          }",
          Some(param),
        ).await.unwrap();

        let mut query_result = ResultParser::new(&result).unwrap();
        let res: Vec<Chat> = query_result.take_array("res").unwrap();
        for msg in res {
          last_date = msg.mdate;
          last_id = msg.id;
          println!("Vous: {}", msg.content);
        }
      }
      _ => {} //on ignore les autres évènements
    }
  }
});
```
Les lignes 42 à 86 definissent la boucle qui va écouter les évenements **Discret**. Elle est lancée dans une tache asynchrone pour que l'application ait ainsi deux boucles indépendantes:
- la boucle de lecture des évènements
- la boucle lisant les messages écrit dans la console 

Lors qu'un évenement de type **DataChanged** est détecté, nous effectuons une requête pour récupérer les données nouvellement recues:
```rust ,linenos,linenostart=58
//...
let result: String = event_app.query(
  "query {
      res: chat.Message(
          order_by(mdate asc, id asc), 
          after($mdate, $id),
          room_id = $room_id
      ) {
              id
              mdate
              content
      }
  }",
  Some(param),
).await.unwrap();
//...
```
Vous pourrez en apprendre plus sur les requêtes de selection de données  dans la section [Requête](@/learn/datamodel/query.fr.md) de la documentation.

Les lignes permettent de lire le résultat JSON de la requête et de les tranformer en liste d'objects *Chat* 
```rust ,linenos,linenostart=73
//...
let mut query_result = ResultParser::new(&result).unwrap();
let res: Vec<Chat> = query_result.take_array("res").unwrap();
//...
```

Si vous lancez ce code dans deux repertoires ou sur deux machines différentes en réseau local, vous pourrez voir les messages s'afficher sur les différentes consoles. 

Félicitation! vous pouvez désormais discuter avec vous même en communicant en *Peer to Peer*! 

# Aller plus loin
Ce tutorial vous a donné un aperçu des bases nécessaires à l'utilisation de **Discret**. Ces connaissances sont suffisantes pour créer des applications ne necessitant la synchronisation qu'avec un seul utilisateur. 

Par exemple, vous pouvez créer un gestionaire de mot de passes qui se synchronisera entre vos différent apprareils en réseau local.

Pour en apprendre plus, vous pouvez suivre les tutoriaux Flutter, en particulier le [chat multi utilisateurs ](@/tutorial/flutter/flutter_multiuser_chat.fr.md) qui introduit la gestion des invitations et la création de *Room*.

La section  [Apprendre](@/learn/_index.fr.md) vous permettra d'approfondir vos connaissances.




# Le code source complet
```rust
use std::{io, path::PathBuf};

use discret::{
  derive_pass_phrase, zero_uid, Configuration, 
  Discret, Parameters, ParametersAdd, ResultParser,
};
use serde::Deserialize;

//identifiant unique de l'application
const APPLICATION_KEY: &str = "github.com/discretlib/rust_example_simple_chat";

#[tokio::main]
async fn main() {
  //definition du modèle de données
  let model = "chat {
    Message{
      content:String
    }
  }";

  let path: PathBuf = "test_data".into(); //où les donnée sont stockées

  //utilisé pour dériver les tous les secrets nécessaires au fonctionnement de Discret
  let key_material: [u8; 32] = derive_pass_phrase("login", "password");

  //démarre Discret 
  let app: Discret = Discret::new(
    model,
    APPLICATION_KEY,
    &key_material,
    path,
    Configuration::default(),
  ).await.unwrap();

  //cette structure permet de désérialiser les messages
  #[derive(Deserialize)]
  struct Chat {
    pub id: String,
    pub mdate: i64,
    pub content: String,
  }

  //écoute les évènements créés par Discret
  let mut events = app.subscribe_for_events().await;
  let event_app: Discret = app.clone();
  tokio::spawn(async move {
    let mut last_date = 0;
    let mut last_id = zero_uid();

    
    let private_room: String = event_app.private_room();
    while let Ok(event) = events.recv().await {
      match event {
        //déclenché quand des données sont insérées ou modifées
        discret::Event::DataChanged(_) => {
          let mut param = Parameters::new();
          param.add("mdate", last_date).unwrap();
          param.add("id", last_id.clone()).unwrap();
          param.add("room_id", private_room.clone()).unwrap();

          //obtiens les dernières données dans le format JSON 
          let result: String = event_app.query(
            "query {
                res: chat.Message(
                    order_by(mdate asc, id asc), 
                    after($mdate, $id),
                    room_id = $room_id
                ) {
                        id
                        mdate
                        content
                }
            }",
            Some(param),
          ).await.unwrap();

          let mut query_result = ResultParser::new(&result).unwrap();
          let res: Vec<Chat> = query_result.take_array("res").unwrap();
          for msg in res {
            last_date = msg.mdate;
            last_id = msg.id;
            println!("Vous: {}", msg.content);
          }
        }
        _ => {} //on ignore les autres évènements
      }
    }
  });

  //les données sont insérées dans votre Room privée
  let private_room: String = app.private_room();
  let stdin = io::stdin();
  let mut line = String::new();
  println!("{}", "Write Something!");
  loop {
    stdin.read_line(&mut line).unwrap();
    if line.starts_with("/q") {
    break;
    }
    line.pop();
    let mut params = Parameters::new();
    params.add("message", line.clone()).unwrap();
    params.add("room_id", private_room.clone()).unwrap();
    app.mutate(
    "mutate {
        chat.Message {
            room_id:$room_id 
            content: $message 
        }
    }",
    Some(params),
    ).await.unwrap();

    line.clear();
  }
}
```
