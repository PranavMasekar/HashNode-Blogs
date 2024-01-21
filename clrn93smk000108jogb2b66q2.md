---
title: "Fireside Unit Testing: Explore testing in Firebase in Dart"
datePublished: Sun Jan 21 2024 08:43:04 GMT+0000 (Coordinated Universal Time)
cuid: clrn93smk000108jogb2b66q2
slug: firebase-testing
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1705826325802/e071ea62-5b87-4ea7-a04e-5ae4f776ecda.png
tags: unit-testing, dart, firebase, flutter, testing

---

# Introduction

Welcome to the third part of our Unit Testing series in Flutter and Dart. Today, we're tackling the challenge of testing Flutter apps that use Firebase for user authentication (`firebase_auth`) and data storage (`cloud_firestore`). Testing these features can be tricky because making real calls to Firebase servers during tests can slow things down and make tests less reliable.

To make our lives easier, we'll explore why it's a great idea to use special packages like `firebase_auth_mocks` and `fake_cloud_firestore`. These packages help us simulate Firebase behavior without actually talking to the Firebase servers, making our tests faster, more predictable, and, most importantly, hassle-free.

## Unit Testing on Firebase Auth

In this blog I am going to use the same QuizGo project for the reference that I used in previous blogs. You can check out the repository [here](https://github.com/PranavMasekar/QuizGo).

```dart
FutureAppEither<String> login(String email, String password) async {
    try {
      final result = await _auth.signInWithEmailAndPassword(
        email: email,
        password: password,
      );

      final user = result.user;
      if (user == null) {
        throw 'Something went wrong!!!';
      }

      return right(user.uid);
    } on FirebaseAuthException catch (error) {
      final messaage = FirebaseHelper.getMessageFromErrorCode(error);
      return left(AppError(message: messaage));
    } catch (e) {
      log('Error Message in SignUp : $e');
      return left(AppError(message: e.toString()));
    }
  }
```

This is the simple login function we need to test. This takes input of email and password and creates the firebase user and returns the unique identifier on success. On failure it returns an AppError instance i.e. just custom error class.

```dart
import 'package:fake_cloud_firestore/fake_cloud_firestore.dart';
import 'package:firebase_auth/firebase_auth.dart';
import 'package:firebase_auth_mocks/firebase_auth_mocks.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mock_exceptions/mock_exceptions.dart';
import 'package:quiz_go/constants/error.dart';
import 'package:quiz_go/services/auth_service.dart';

void main() {
  const email = 'someone@gmail.com';
  const password = '123456';

  late final MockFirebaseAuth mockFirebaseAuth;
  late final FakeFirebaseFirestore fakeFirebaseFirestore;
  late final AuthService authService;

  setUpAll(() {
    // Created mock FirebaseAuth class
    mockFirebaseAuth = MockFirebaseAuth();
    // Created mock FirebaseFirestore class
    fakeFirebaseFirestore = FakeFirebaseFirestore();
    authService = AuthService(
      auth: mockFirebaseAuth,
      firestore: fakeFirebaseFirestore,
    );
  });

  group(
    'Login Function -',
    () {
      test(
        'should return userID on success',
        () async {
          final result = await authService.login(email, password);

          final isRight = result.isRight();
          expect(isRight, true);

          final uid = result.toNullable();
          expect(uid, isA<String>());

          final error = result.getLeft().toNullable();
          expect(error, null);
        },
      );

      test(
        'should return AppError on exception',
        () async {
          // Changing behavior of signInWithEmailAndPassword method
          whenCalling(
            Invocation.method(#signInWithEmailAndPassword, null),
          )
              .on(mockFirebaseAuth)
              .thenThrow(FirebaseAuthException(code: 'Random Error'));

          expect(
            () => mockFirebaseAuth.signInWithEmailAndPassword(
              email: email,
              password: password,
            ),
            throwsA(isA<FirebaseAuthException>()),
          );

          final result = await authService.login(email, password);

          final isLeft = result.isLeft();
          expect(isLeft, true);

          final error = result.getLeft().toNullable();
          expect(error, isA<AppError>());
        },
      );
    },
  );
}
```

### Here’s what happening in the code

`setUpAll`, is executed once before any tests run. It initializes mock instances of Firebase authentication (MockFirebaseAuth) and Firestore (FakeFirebaseFirestore). Additionally, it creates an instance of the AuthService class, injecting the mock authentication and Firestore instances. This ensures that each test operates in a controlled environment.

First test checks the login method of the AuthService class. It verifies that a successful login operation returns a result indicating success (isRight), a non-null user ID (uid), and no errors (error is null). Because we used the mocked version of firebase auth class, we don’t need to configure the actual mocking of functions. 

Second test simulates an exception during the login process by throwing a FirebaseAuthException. It first ensures that the exception is properly thrown when attempting to sign in with mockFirebaseAuth. Then, it checks that the login method of AuthService returns an AppError in case of an exception.

> Note : #signInWithEmailAndPassword is a symbol representing the name of the method being invoked, in this case, the signInWithEmailAndPassword method of FirebaseAuth. `null` indicates that there are no arguments passed to this method call.

## Unit Testing on Firebase Firestore

```dart
FutureAppEither<bool> saveToFirestore(Map<String, dynamic> data) async {
    try {
      final uid = _auth.currentUser!.uid;
      await _firestore.collection('users').doc(uid).set(data);
      return right(true);
    } catch (error) {
      log('Error Message in saveToFirestore : $error');
      return left(
        AppError(message: 'Failured in updating user'),
      );
    }
  }
```

This is a simple function which sets users' documents in the firestore. In actual application flow this function will always be called after login or signup functions.

```dart
import 'package:fake_cloud_firestore/fake_cloud_firestore.dart';
import 'package:firebase_auth/firebase_auth.dart';
import 'package:firebase_auth_mocks/firebase_auth_mocks.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mock_exceptions/mock_exceptions.dart';
import 'package:quiz_go/constants/error.dart';
import 'package:quiz_go/services/auth_service.dart';

void main() {
  const email = 'someone@gmail.com';
  const password = '123456';
  const userName = 'Someone';

  late final MockFirebaseAuth mockFirebaseAuth;
  late final FakeFirebaseFirestore fakeFirebaseFirestore;
  late final AuthService authService;

  setUpAll(() {
    mockFirebaseAuth = MockFirebaseAuth();
    fakeFirebaseFirestore = FakeFirebaseFirestore();
    authService = AuthService(
      auth: mockFirebaseAuth,
      firestore: fakeFirebaseFirestore,
    );
  });

  group(
    'saveToFirestore -',
    () {
      test(
        'should return true after saving data',
        () async {
          await authService.signUp(email, password, userName);
          final result = await authService.saveToFirestore(
            {'email': email, 'userName': userName},
          );

          final isRight = result.isRight();
          expect(isRight, true);

          final isSuccess = result.toNullable();
          expect(isSuccess, true);

          final error = result.getLeft().toNullable();
          expect(error, null);
        },
      );

      test(
        'should return AppError on exception',
        () async {
          await authService.signUp(email, password, userName);
          final uid = mockFirebaseAuth.currentUser!.uid;
          // Getting fake document from Firestore
          final doc = fakeFirebaseFirestore.collection('users').doc(uid);
          whenCalling(
            Invocation.method(#set, null),
          ).on(doc).thenThrow(FirebaseException(plugin: 'firestore'));

          expect(
            () => doc.set(
              {'email': email, 'userName': userName},
            ),
            throwsA(isA<FirebaseException>()),
          );

          final result = await authService.saveToFirestore(
            {'email': email, 'userName': userName},
          );

          final isLeft = result.isLeft();
          expect(isLeft, true);

          final error = result.getLeft().toNullable();
          expect(error, isA<AppError>());
        },
      );
    },
  );
}
```

First test case checks the `saveToFirestore` method in the AuthService class. It first signs up a user using the signUp method, and then it attempts to save data to Firestore using `saveToFirestore`. The test asserts that the result is a success (isRight), the returned value is true (isSuccess), and there are no errors (error is null).

Second test case simulates an exception during the Firestore data-saving process. It sets up a mock behavior for the set method on a Firestore document (doc). The test then asserts that attempting to set data on the document throws a FirebaseException. Finally, it checks that the `saveToFirestore` method in AuthService returns an AppError in case of an exception.

Congratulations on completing the third part of our Flutter Testing Series! Keep up the excellent work, and may your testing endeavors continue to elevate your Flutter development skills. Cheers to your testing success!

Keep Fluttering 💙💙💙