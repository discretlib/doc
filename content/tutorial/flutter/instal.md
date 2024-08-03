+++
title = "Installing Components"
description = "Learn how to setup the Flutter binding"
weight = 0
+++
# Platform Support

# Platform Support
- Linux: Tested 
- Windows: Tested 
- macOS: not tested, should work
- Android: works on arch64 architecture. Architectures i686 and x86_64 have some low level linker issues when working with Flutter.
- iOS: not tested

# Installing Components

To get started, you need to have [Flutter SDK](https://docs.flutter.dev/get-started/install) and [Rust toolchain](https://www.rust-lang.org/tools/install) installed on your system.

If you're working on Linux, **do not** install Flutter from **snap**. Flutter from snap comes with its own binary linker called ld, which is currently incompatible with Rust. Instead, follow the manual installation method as written in the Flutter docs.

Ensure that your rust version is greater or equal to *1.79.0*

Once you're done with the installations, verify your system's readiness with the following commands. 

```
rustc --version
flutter doctor
```

# Getting the binding.
The binding is not provided as a library but as a complete Flutter project, if you have ideas on how to provides a simpler binding you are welcome to contribute!

To install the binding clone the following repository:
```
cd your_dev_repository
git clone https://github.com/discretlib/discret_flutter.git
```

# Building and Running a Sample application

To test the installation you can rename one of the provided example into main.dart and run the application.

```
cp ./lib/example_simple_chat.dart ./lib/main.dart
flutter run
```

The first run will need to download all rust dependencies and compile them, wich can take a few minutes.
