# Dependency Injection

Hello everyone, in this article we are going in-depth with the Flutter GetIt package. We will understand why we use this package. What does it provide? What problem does it solve? and many more.

In a Flutter application, it can be challenging to manage dependencies and objects that need to be shared across different parts of the application. Hardcoded object references and global variables can lead to code that is difficult to maintain, test, and extend. Moreover, creating singletons and manually managing object lifecycles can be time-consuming and error-prone. We need a solution that can simplify the process of accessing objects and dependencies in a Flutter application while promoting code modularity and testability.

To solve these dependency problems we use a package called GetIt. GetIt is a simple service locator for Dart and Flutter projects.

In this article, we are going to see how to use the GetIt package in our flutter project. First, let’s import the package - Go to pubspec.yaml file and add the latest version of the [GetIt package](https://pub.dev/packages/get_it).

get\_it: ^7.2.0

Now to create a locator.dart file in the lib folder i.e. same position as the main.dart file. We are going to inject a simple Auth Bloc which requires an instance of Auth Service.

If you don’t have an idea about BLoC then please read my other articles regarding Flutter BLoC.

Assuming you have your Auth BLoC and Auth service ready, let’s start writing our locator.dart file.

```dart
import 'package:get_it/get_it.dart';
final locator = GetIt.instance;

Future<void> init() async {
   locator.registerFactory(() => AuthBloc(authService: locator()));
   locator.registerLazySingleton(() => AuthService());
 }
```

First, we create an instance of GetIt using GetIt.instance.

Then we create a method called init in which we will register our dependencies.

Now let’s take a deep dive into the function provided by the locator - 

  - Here assigning locator() indicates that GetIt will automatically assign the desired parameter provided that we have registered that dependency in the locator file.

In short, assigning locator() to the authService parameter in Auth BLoC will provide the required dependency.

  - **registerFactory** is a function to register dependency which can have multiple instances. In our case, it’s going to be our Auth BLoC.

  - **registerLazySingleton** is a function to register dependency which can only have one instance i.e Singleton Class. In our case, it's going to be our Auth Service. Instead of manually creating a singleton class we can use this simple function to register a singleton.

Now we have just created our init function but we need to call it somewhere right? 

As we want our dependencies to be available as soon as the app starts we will call this init method just above runApp method.

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await init();
  runApp(const MyApp());
}
```

Now, whenever we want to use a registered service we will use this -

```dart
body: BlocProvider(
     create: (context) => locator<AuthBloc>(),
     child: child,
  ),
```

I hope this blog will provide you with sufficient information on Trying up the GetIt package in your flutter projects. We showed you what the GetIt package is. its properties of the GetIt package. We made a demo program for the GetIt package. So please try it.