+++
title = "Installer les composants"
description = "Comment installer et tester l'API rust"
weight = 0
+++
# Platform Support
- Linux: testé
- Windows: testé
- macOS: non testé, devrait fonctioner
- Android: Fonctionne sur architecture arch64. Les architectures i686 and x86_64 ont des problèmes liés au *linker*
- iOS: non testé


# Installing Components

Avant de commencer, sous devez avoir installé le [Flutter SDK](https://docs.flutter.dev/get-started/install) et [Rust](https://www.rust-lang.org/tools/install). 


Si vous travaillez sous Linux, **n'installez pas** Flutter avec **snap**. Le **snap** Flutter contient un linker appelé ld qui n'est pas compatible avec Rust. A la place, installez Flutter manuellement Flutter en suivant la documentation.



Vous devez vous assurer que la version de Rust est supérieure ou égale à **1.79.0**

Vous pouvez vérifier l'état des deux installations avec les commandes
```
rustc --version
flutter doctor
```

# Récupérer le *binding* Flutter

Le *binding* Flutter n'es pas une librairie mais un projet Flutter complet.

Pour le récupérer, clonez le projet suivant:
```
cd your_dev_repository
git clone https://github.com/discretlib/discret_flutter.git
```

# Tester l'installation

Pour tester que tout fonctionne, vous pouvez lancer un des exemples fournit.
Copiez un des exemple dans le fichier ./lib/main.dart et lancer le avec flutter
```
cp ./lib/example_simple_chat.dart ./lib/main.dart
flutter run
```

Le premier lancement va télécharger toutes les dépendances requises et les compiler. Cela peut prendre plusieurs minutes.
