---
title: "Simplifying Navigation in Flutter with the GoRouter Package"
datePublished: Mon Apr 03 2023 18:09:45 GMT+0000 (Coordinated Universal Time)
cuid: clg15byob000b09mm9ogw5c9x
slug: gorouter
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1680544851127/339e3e1c-e4f2-4939-8046-4b43947f179a.jpeg
tags: dart, flutter, wemakedevs

---

**Introduction**:

The GoRouter package in Flutter is a routing library that simplifies the navigation process in your Flutter application. It provides an intuitive API for defining routes and navigating between them, making it easier for developers to manage their app's navigation flow. With GoRouter, you can create complex navigation flows, such as nested or dynamic routes, with ease.

In this hands-on blog, we'll take a closer look at the GoRouter package and explore how to use it in a Flutter application. We'll cover the following topics:

* Installing the GoRouter package
    
* Defining routes using GoRouter
    
* Navigating between routes
    
* Passing data between routes
    
* Handling unknown routes
    
* Nested routes
    

By the end of this blog, you should have a good understanding of how to use the GoRouter package to create a robust and efficient navigation system in your Flutter application. In this blog, I am going to use one simple application i.e. [music app.](https://github.com/PranavMasekar/Music-App/tree/Riverpod)

Before going to actual implementation let’s first clear the concepts related to the GoRouter package. Suppose you are building an E-Commerce application where you have a screen displaying a list of products and one screen showing details of individual products. So our routing scheme is like /products for all products and /products/${product\_id} for individual products. But creating such types of nested routes for large applications can become a mess in basic flutter navigation. And if you are even making a web application using Flutter then this navigation becomes more crucial as websites can be accessed by direct URLs. Other than this using GoRouter we can handle error routes. GoRouter is designed to be lightweight and efficient, which means it can help improve the performance of your application, especially when dealing with complex routing logic.

Let's get started!

**Installation :** 

Add the following line to pubspec.yaml file and run flutter pub get command - 

`go_router: ^6.5.2`

**Implementation :** 

First, let’s create a router.dart file which will contain all of our routes and routing logic. 

```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';
import 'package:music_app/route_names.dart';
import 'package:music_app/view/home.dart';
import 'package:music_app/view/initial.dart';
import 'package:music_app/view/song_details_page.dart';

import 'view/errorpage.dart';

class AppRouter {
  GoRouter router = GoRouter(
    initialLocation: "/",
    routes: [
      GoRoute(
        name: RouteNames.inital,
        path: '/',
        pageBuilder: (context, state) {
          return MaterialPage(child: InitialScreen());
        },
      ),
      GoRoute(
        name: RouteNames.home,
        path: '/song',
        pageBuilder: (context, state) {
          return MaterialPage(child: HomeScreen());
        },
        routes: [
          GoRoute(
            name: RouteNames.songDetail,
            path: ':id',
            pageBuilder: (context, state) {
              return MaterialPage(
                child: SongDetails(
                  id: int.parse(state.params["id"]!),
                ),
              );
            },
          ),
        ],
      ),
      GoRoute(
        name: RouteNames.error,
        path: '/error',
        pageBuilder: (context, state) {
          return MaterialPage(
            child: ErrorPage(errorMessage: state.queryParams["errorMsg"]!),
          );
        },
      ),
    ],
    errorPageBuilder: (context, state) {
      return MaterialPage(child: ErrorPage(errorMessage: "Wrong route"));
    },
  );
}
```

We need to first create the object of the GoRouter class and for this object, we have to pass it in the main.dart file, so let’s create the router here. InitialLocation defines the starting route for the application it shouldn’t always be “/”, it can be anything but standard practice is to give “/” as the default home route. After that, we define our named routes. We define a router using the GoRoute class which requires a name, path and pageBuilder function. Name is the identity of the router, the path is the actual URL structure and PageBuilder is a function that returns a page. PageBuilder comes with two parameters one is context and another state, state gives us access to params and queryParams. 

**params**: Like in an HTTP request we pass some id in a URL like `https://sample.app/produts/RMX-2170` here RMX-2170 is the id of the product. Similar to this when we use the params to get the information directly from the URL. This greatly helps in flutter web and also in deep linking of the application.

**queryParams**: QueryParams are like the body of an HTTP request where we pass the data with the request but that isn’t part of the URL. If we want to send some data in the route secretly not by exposing it to a URL then we use queryParams.

Something that needs to be noticed in the code is how we set up the nested routing.

The GoRoute class also contains a list of routes. So, in this case, the parent route is /song and inside the parent route, we want a child route /song/:id. Similarly, we can have more nesting of routes according to our needs.

We also have a special method called errorPageBuilder which gets automatically triggered if any unknown route has been pushed into the stack. 

After making our router we need to add this to the main.dart file. We have to modify the existing MaterialApp to MaterialApp.router() to add the GoRouter Functionality.

Now let’s see how to use the GoRouter in action - 

We can navigate by using context.goNamed() function which takes the route name as input and some optional parameters like params and queryParams.

```dart
    context.goNamed(
            RouteNames.songDetail,
            params: {
                "id": song.songId.toString(),
             },
          );
```

Similarly for queryParams 

```dart
context.goNamed(
    RouteNames.error, 
    queryParams: {
        "errorMsg": "Something went wrong"
        }
    );
```

**Congratulations** 🥳 🥳 You just integrated GoRouter Navigation in your flutter project. By implementing the GoRouter package in your Flutter application, you can create a smoother and more seamless user experience while simplifying the management of your app's navigation flow.