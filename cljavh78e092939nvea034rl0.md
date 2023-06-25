---
title: "Mastering State Management in Flutter with Riverpod : Part - 1"
datePublished: Sun Jun 25 2023 03:30:42 GMT+0000 (Coordinated Universal Time)
cuid: cljavh78e092939nvea034rl0
slug: riverpod-part-1
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1687359480160/886f1316-8390-47e2-8755-e1bd5bf397a4.png
tags: flutter, state-management, dependency-management, wemakedevs, riverpod

---

Welcome to our tech blog, where we delve into the fascinating world of Flutter and its advanced state management solutions. If you're a Flutter developer seeking to streamline your app's state management, you've come to the right place. In this comprehensive series, we will guide you through the ins and outs of Riverpod, a powerful and versatile state management library that has taken the Flutter community by storm. This is the first part of the Riverpod series. So, let's begin our adventure and unlock the full potential of Riverpod together!!!

### What is Riverpod?

[Riverpod](https://riverpod.dev/) is a state management library for Flutter, designed to simplify and enhance the way you manage states in your applications. Developed by the same team behind the popular Provider package, Riverpod takes the concept of dependency injection and combines it with a powerful, reactive state management system. At its core, Riverpod provides a way to manage the state of your app by using providers. Providers are objects that hold and expose data or services to other parts of your application. Whether you're working on a small personal project or a large-scale application, Riverpod offers the flexibility and power to handle complex state management scenarios with ease.

### Riverpod Concepts -

By including the flutter\_riverpod package we get a lot of widget types and providers.

* **Ref**:- In Riverpod, the ref object is a crucial component that allows you to interact with providers and perform various operations within the provider system. The best analogy for a ref is context, as context is used to interact with widgets ref is used to interact with providers.
    
* **ProviderScope**:-  In Riverpod, provider scope refers to the visibility and accessibility of a provider within the widget tree. We used this function above runApp() method to access our providers from anywhere.
    
* **ConsumerWidget**:- ConsumerWidget is an alternative for StatlessWidget of Flutter. It works the same as a stateless widget but gives us an extra parameter in the build method i.e. ref.
    
* **ConsumerStatefulWidget**:- ConsumerStatefulWidget is an alternative to Statfulwidget of Flutter. As in StatefulWidget, we have access to Buildcontext anywhere inside the class. The same ConsumerStatefulWidget gives access to ref in the entire class.
    

### **Types of Providers -**

1. **Provider**:- Provider is great for accessing dependencies that don't change, such as the repositories in our app. Where we can use this to access our repository instance from anywhere in the application.
    

```dart
final userRepositoryProvider = Provider<UserRepository>((ref) {
  return UserRepository();
});

Class UserRepository {
    ... // Some Implementation
}
```

We can access this instance using the -

```dart
final userRepository = ref.watch(userRepositoryProvider);
```

This can be used as dependency management of the application where we can use one instance repeatedly.

1. **StateProvider**:- StateProvider is great for storing simple state objects that can change, such as a boolean value for loading.
    

```dart
final isLoadingProvider = StateProvider<bool>((ref) {
  return false;
});
```

We can update the value of StateProvider using ref.read() -

```dart
class UserRepository {
Future<void> getData()async {
	    ref.read(isLoadingProvider.notifier).state = true;
        . . . [Some Code]
        ref.read(isLoadingProvider.notifier).state = false;
    }
}
```

And we can read the value using the -

```dart
final isLoading = ref.watch(isLoadingProvider);
```

1. **FutureProvider**:- FutureProvider is mainly used when we are dealing with async calls like API calls. FutureProvider offers flexibility to handle errors as well.
    

```dart
final loginProvider = FutureProvider<User>((ref) {
  final userRepository = ref.watch(userRepository);
  return userRepository.login(); // Async Function Call
});
```

We can use this as -

```dart
Widget build(BuildContext context, WidgetRef ref) {
  final login = ref.watch(loginProvider);
  return login.when(
    loading: () => const CircularProgressIndicator(),
    error: (err, stack) => Text('Error: $err'),
    data: (user) => Text(user.name),
  );
}
```

1. **StreamProvider**:- We use StreamProvider to watch a Stream of results from a real-time API and reactively rebuild the UI. Here is the classic example of a Firebase login check -
    

```dart
final authStateChangeProvider = StreamProvider.autoDispose<User?>((ref) {
  final firebaseAuth = ref.watch(firebaseAuthProvider);
  return firebaseAuth.authStateChanges();
});
```

```dart
Widget build(BuildContext context, WidgetRef ref) {
  final authStateChange = ref.watch(authStateChangeProvider);
  return authStateChange.when(
    data: (user) {},
    loading: () => const CircularProgressIndicator(),
    error: (err, stack) => Text('Error: $err'),
  );
}
```

1. **StateNotifierProvider**:- We use StateNotifierProvider when we want to use a NotifierClass with Riverpod. Suppose we have -
    

```dart
class CounterNotifier extends StateNotifier<int> {
  CounterNotifier() : super(0);
  void increment() {
    state++;
  }
}

final counterProvider = StateNotifierProvider<CounterNotifier>((ref) {
  return CounterNotifier();
});
```

We read this provider by -

```dart
final counter = ref.watch(counterProvider);
```

And update this provider -

```dart
ref.read(counterProvider.notifier).increment();
```

> Remember this is just an example. StateNotifier can be used to handle very complex objects and classes.

1. **ChangeNotifierProvider**:- We use ChangeNotifierProvider when we are using old provider classes which extend ChangeNotifier.
    

```dart
class Counter extends ChangeNotifier {
  int count = 0;
  void increment() {
    count++;
    notifyListeners();
  }
}

final counterProvider = ChangeNotifierProvider<Counter>((ref) {
  return Counter();
});
```

We can read the provider as the same -

```dart
final counter = ref.watch(counterProvider);
```

And call any function inside the provider using the -

```dart
 ref.read(counterProvider).increment();
```

**Congratulations** on reaching the end of this blog! You've now gained a solid understanding of Flutter Riverpod for state management. We covered the fundamental concepts of Riverpod, including providers, state management, and the role of the ref object. We explored all the different types of providers in Riverpod. In the next part, we are going to build the entire application using Riverpod, Until then

Keep Fluttering 💙💙💙