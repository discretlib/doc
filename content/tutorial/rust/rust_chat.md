+++
title = "Minimalist Rust Chat"
description = "learn how to create a minimal Chat application using Rust"
weight = 1
+++

In this document, we will create a minimalistic Chat application that will allow you to discuss with yourself. While not super useful, it introduces the different concept used by **Discret**

Nous allons créer un application de chat vraiment minimaliste qui ne permettra de discuter qu'avec vous même. Bien que peut utile, cela permet d'introduire les différents mécanismes utilisés par *Discret*.

# Setup
Let 's start by creating a new Rust project named  *rust_simple_chat*
```
cargo new rust_simple_chat
cd rust_simple_chat
```

Open the **Cargo.toml** file to add the required dependencies:
```toml
[dependencies]
    discret = "0.6.0"
    tokio = { version = "1.38.0", features = ["full"] }
    serde = { version = "1.0.203", features = ["derive"] }
    serde_json = "1.0.117"
```

# *Discret* Initialisation

*Discret* requires some parameters: 
- **a data model**: defines the kind of data that can be used in *Discret*,
- **an application unique identifier**: once the application in production, this identifier cannot be changed anymore. It is used to derive some secrets that will be used by *Discret*. If this identifier is changed, the secrets will change and users will not be able to connect anymore.
- **a user master secret**: this per user secret is used with the application identifier to create the user secrets. In this this example we will use a password derivation function to create this secret.
-  **a data folder**: where data is stored.
-  **A configuration**: we will use the default configuration.


Replace **src/main.rs** default file by this one: 
```rust,linenos
use std::{io, path::PathBuf};

use discret::{
  derive_pass_phrase, zero_uid, Configuration, 
  Discret, Parameters, ParametersAdd, ResultParser,
};
use serde::Deserialize;

//application unique identifier
const APPLICATION_KEY: &str = "github.com/discretlib/rust_example_simple_chat";

#[tokio::main]
async fn main() {
  //the data model
  let model = "chat {
    Message{
      content:String
    }
  }";

  let path: PathBuf = "test_data".into(); //where data is stored

  //the user master secret
  let key_material: [u8; 32] = derive_pass_phrase("login", "password");

  //starts Discret 
  let app: Discret = Discret::new(
    model,
    APPLICATION_KEY,
    &key_material,
    path,
    Configuration::default(),
  ).await.unwrap();
}
```
Line 10 defines the application unique identifier:
```rust,linenos,linenostart=9
//...
const APPLICATION_KEY: &str = "github.com/discretlib/rust_example_simple_chat";
//...
```
 
---

 Lines 15 to 19 define the data model used by this application. It creates a namespace **chat** that contains an Entity names **Message**, which contains an unique field named **content** of the String type. 

You can learn more about data models in the documentation [Schema and Entities](@/learn/datamodel/schema.md).
```rust,linenos,linenostart=14
//...
  let model = "chat {
    Message{
      content:String
    }
  }";
//...
```

---
 
Line 24 creates the user master secret. It will be used by Discret to create:
- the database encryption key. All data is encrypted at rest.
- A signature/verifying key that will be used to identify the user to other peers and sign data created and modified by this user.
  
The **derive_pass_phrase**  uses the Argon2id  key derivation function with the parameters recommended by [owasp.org](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)

```rust,linenos,linenostart=23
//...
let key_material: [u8; 32] = derive_pass_phrase("login", "password");
//...
```


# Insert Messages

Once Discret initialized, we can insert new data. This example will read the messages from the console and write them into the Discret database. 


Insert the following lines after line 33 **).await.unwrap()**;

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

Line 34 retrieves the user's private *Room*. *Rooms* are the access rights system used to synchronized data with other peers. Data inserted in a *Room* is synchronized only with peers that have access rights to it.

In this example we will only use the user's private *Room*, meaning that only him will able to access the data. If this user's is connected on several devices, data will be synchronized between  devices.

You can learn more about access right in the [Room](@/learn/access_rights/room.md) section of the documentation.

---

Lines 47 to 57 defines the query used to insert messages:
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

This is a **mutation** query to insert a new tuple of the **chat.Message** entity defined in the data model.

You can notice that the **room_id**  has not been manually defined in the data model. It is a system field available for every entities. **\$room_id** and **\$message** are query parameters that are passed by the **Some(params)** object.


You can lean more about data insertion and modification in the [Mutations and Deletions](@/learn/datamodel/mutation.md) section of the  documentation.

If you run the application, you should be able to write messages.
```
cargo run
```

# Read synchronized data

At this point, inserted data will be synchronized between each instance of the application. 

To retrieve the synchronized messages, we will listen to the events sent by *Discret* and read messages from the database when we receive an event saying that new data is available.

Insert the following code between  ligne 33 **).await.unwrap();** and line 34: **let private_room: String = app.private_room();**

```rust ,linenos,linenostart=33
//this struct is used to parse the query result
#[derive(Deserialize)]
struct Chat {
  pub id: String,
  pub mdate: i64,
  pub content: String,
}

 //listen for events
let mut events = app.subscribe_for_events().await;
let event_app: Discret = app.clone();
tokio::spawn(async move {
  let mut last_date = 0;
  let mut last_id = zero_uid();

  let private_room: String = event_app.private_room();
  while let Ok(event) = events.recv().await {
    match event {
       //triggered when data is inserted or modified
      discret::Event::DataChanged(_) => {
        let mut param = Parameters::new();
        param.add("mdate", last_date).unwrap();
        param.add("id", last_id.clone()).unwrap();
        param.add("room_id", private_room.clone()).unwrap();

         //get the latest data in the JSON format 
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
      _ => {}  //ignores other events
    }
  }
});
```

Lines 42 to 86 
Les lignes 42 à 86 define the loop that will listen for the *Discret* events. it is launched as an asynchronous task to avoid blocking the application.

When a **DataChanged** event is triggered, we query the database to retrieve the new data:

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
You can learn more about reading database data in the [Query](@/learn/datamodel/query.md) section of the documentation.

---
Lines 74 and 75 parse the JSON result to transform it into a list of *Chat* object.
```rust ,linenos,linenostart=73
//...
let mut query_result = ResultParser::new(&result).unwrap();
let res: Vec<Chat> = query_result.take_array("res").unwrap();
//...
```

Now, if you run this example in different folders or different devices on your local network, you should be able to chat with yourself!

# Going Further

This tutorial provided the basis to use *Discret*, you should now be able to create applications that requires only one user to be synchronized. For example you could create a password manager that synchronize data across your devices.

To learn more about discret you can follow the Flutter tutorials. [Flutter chat with multiple users](@/tutorial/flutter/flutter_multiuser_chat.md) creates a multi user chat and introduces you to *Room* creations and how to create Invitations to be sent to other peers. 

The [Learn](@/learn/_index.md) section will provide deeper explanations on how to use *Discret*



# Full Source Code 
```rust
use std::{io, path::PathBuf};

use discret::{
  derive_pass_phrase, zero_uid, Configuration, 
  Discret, Parameters, ParametersAdd, ResultParser,
};
use serde::Deserialize;

//application unique identifier
const APPLICATION_KEY: &str = "github.com/discretlib/rust_example_simple_chat";

#[tokio::main]
async fn main() {
  //defines the datamodel
  let model = "chat {
    Message{
      content:String
    }
  }";


  let path: PathBuf = "test_data".into(); //where data is stored

  //used to derives all necessary secrets
  let key_material: [u8; 32] = derive_pass_phrase("login", "password");

  //starts Discret
  let app: Discret = Discret::new(
    model,
    APPLICATION_KEY,
    &key_material,
    path,
    Configuration::default(),
  ).await.unwrap();

  //this struct is used to parse the query result
  #[derive(Deserialize)]
  struct Chat {
    pub id: String,
    pub mdate: i64,
    pub content: String,
  }

  //listen for events
  let mut events = app.subscribe_for_events().await;
  let event_app: Discret = app.clone();
  tokio::spawn(async move {
    let mut last_date = 0;
    let mut last_id = zero_uid();

    
    let private_room: String = event_app.private_room();
    while let Ok(event) = events.recv().await {
      match event {
        //triggered when data is inserted or modified
        discret::Event::DataChanged(_) => {
          let mut param = Parameters::new();
          param.add("mdate", last_date).unwrap();
          param.add("id", last_id.clone()).unwrap();
          param.add("room_id", private_room.clone()).unwrap();

          //get the latest data in the JSON format 
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
            println!("you said: {}", msg.content);
          }
        }
        _ => {} //ignores other events
      }
    }
  });

  //Message are inserted in your private room
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