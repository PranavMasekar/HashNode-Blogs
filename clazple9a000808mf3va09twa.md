# Flutter BLoC Beginner's Guide Part - 1

State management is one of the hot topics when it comes to Flutter. There is no doubt that handling a state application like a pro will increase the app's performance.  
So when it comes to Flutter we have many options as State management tools, one of them is BLoC. In this blog, we are gonna go through the basics of the bloc and this blog will be part of the Bloc Series.

**What is BLoC?**  
BLoC is a popular state management solution in Flutter app development. BLoC allows us to listen to events occurring inside the application and take appropriate actions on it.

![bloc.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1669560439036/jKi8Vg3zm.png align="center")

This is a famous diagram representing Flutter BLoC.  
Flutter BLoC consists of main two things events and states. Events are actions that are performed by the user, by handling these events we release appropriate states.

**Example :**  
The user clicks on a login button in applications so we call it a Login event. Now we need to handle this event we apply our logic and we conclude that the user is authentic now we need to notify the user that he is authenticated and can move forward in applications. To do that we need to release a state called as UserAuthenticatedState. Suppose the user's credentials were wrong and authentication failed we also need to notify a user about this so we will release a state called as UserAuthenticationFailedState.

Note: The names of states and events are user-defined.

This was the basic flow of how the bloc handles an event in applications.  
To use Bloc in your flutter application import the following packages :  
flutter\_bloc: ^8.1.1  
equatable: ^2.0.5

To know more about the Equatable package refer to this [link](https://sungod.hashnode.dev/flutter-equatable).

So here we are gonna design a simple bloc that authenticates users based on credentials provided by the user.

So we are going to have three files:  
AuthBloc.dart  
AuthEvent.dart  
AuthState.dart

First, we will write our AuthEvent.dart file

AuthEvents contain events that will be fired by users while interacting with the applications. Such events are user click on the login button, So we will fire an event called as LoginEvent which contains two parameters email and password as we need these values to authenticate the user.

```dart
part of 'auth_bloc.dart';

abstract class AuthEvent extends Equatable {
  const AuthEvent();

  @override
  List<Object> get props => [];
}

class LoginEvent extends AuthEvent {
  final String emailId, password;
  LoginEvent({
    required this.emailId,
    required this.password,
  });
  @override
  List<Object> get props => [emailId, password];
}
```

We create a base abstract class named as AuthEvent which is extending Equatable we do this to reduce the repetition of code, which will be generated due to equatable.

Now we will write our AuthState.dart file  
This file contains all the possible states which can occur in the application.

```dart
part of 'auth_bloc.dart';

abstract class AuthState extends Equatable {
  const AuthState();

  @override
  List<Object> get props => [];
}

class AuthInitial extends AuthState {}

class AuthLoadingState extends AuthState {}

class AuthenticatedState extends AuthState {}

class AuthErrorState extends AuthState {
  final String errorMsg;
  AuthErrorState({required this.errorMsg});
  @override
  List<Object> get props => [errorMsg];
}
```

AuthLoadingState indicated that the application is under loading state, this helps us to show loading indicators to the UI.  
Authenticated indicates that the user was authenticated successfully.  
AuthErrorState contains an errorMsg which needs to display to the user if any error occurs.

Now we will write our main AuthBloc.dart file which contains all business logic regarding authentication.  
In this class, we will handle our events and release states.

```dart
import 'dart:developer';

import 'package:bloc/bloc.dart';
import 'package:equatable/equatable.dart';

part 'auth_event.dart';
part 'auth_state.dart';

class AuthBloc extends Bloc<AuthEvent, AuthState> {
  AuthBloc() : super(AuthInitial()) {
    on<LoginEvent>(_handleLoginEvent);
  }

  Future<void> _handleLoginEvent(LoginEvent event, Emitter emit) async {
    emit(AuthLoadingState());
    try {
      // Logic of authentication
      // You can access variables from event as
      log(event.emailId);
      log(event.password);
    } catch (e) {
      emit(AuthErrorState(errorMsg: e.toString()));
    }
  }
}
```

emit() is used to emit states from a bloc.

Now as we have prepared the bloc we will integrate that into our UI.

While adding a particular bloc to the application we need to consider the scope of our bloc. So we will declare our bloc at the root of our application.

```dart
import 'package:bloc_test/bloc/auth_bloc.dart';
import 'package:bloc_test/home_page.dart';
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});
  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (context) => AuthBloc(),
      child: MaterialApp(
        theme: ThemeData(primarySwatch: Colors.blue),
        home: const HomePage(),
      ),
    );
  }
}
```

As we have declared the bloc now it's time to use the bloc.  
In onTap property of the button, we need to add an event called LoginEvent, in order to do that we use BlocProvider.of&lt;\[BLoC Name\]&gt;(context).add(\[Event Name\]);

This will trigger the bloc with LoginEvent and the first loading state will be released.

```dart
BlocListener<AuthBloc, AuthState>(
            listener: (context, state) {
              if (state is AuthErrorState) {
                ScaffoldMessenger.of(context).showSnackBar(
                  SnackBar(content: Text(state.errorMsg)),
                );
              }
              if (state is AuthenticatedState) {
                ScaffoldMessenger.of(context).showSnackBar(
                  const SnackBar(content: Text("LoggedIn")),
                );
              }
            },
            child: TextButton(
              onPressed: () {
                BlocProvider.of<AuthBloc>(context).add(
                  LoginEvent(
                    emailId: emailController.text,
                    password: passwordController.text,
                  ),
                );
              },
              child: const Text("LogIn"),
            ),
          ),
```

Now by using listen to property we are to listen to states being released by the bloc and we will navigate according to state.

That's it, Congratulations you have created your first bloc application. Next in this series, we are going to look into the details of the Equatable package and BLoC wrappers like provider, listener and consumer.