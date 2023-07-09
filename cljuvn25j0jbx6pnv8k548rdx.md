---
title: "Mastering State Management in Flutter with Riverpod : Part - 2"
datePublished: Sun Jul 09 2023 03:30:39 GMT+0000 (Coordinated Universal Time)
cuid: cljuvn25j0jbx6pnv8k548rdx
slug: riverpod-part-2
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1688223160884/0621412a-a6f4-4557-afee-b159b65ed720.png
tags: flutter, state-management, flutter-examples, wemakedevs, riverpod

---

Welcome to our tech blog, where we delve into the fascinating world of Flutter and its advanced state management solutions. In the last part of the Riverpod series, we covered the basics of Riverpod and providers in Riverpod. In this blog, we are going to build a complete Flutter application using Riverpod. So if you haven’t read the first part, I highly recommend it. The application source code is on [GitHub](https://github.com/PranavMasekar/Music-App/tree/Riverpod).

So we are going to build a music application which has three screens. First, we have a screen where we list down all the songs, Second we have song details screen where we show lyrics and last one is the error screen. This is the same application we built in the Flutter BLoC series. In this project, I have used GoRouter for the navigation which I have already covered in a separate blog. We are not going to focus on building UI and navigation in this blog. We are only going to focus on State management-related things. So let’s first import the necessary packages -

```yaml
flutter_riverpod: ^2.0.2
fpdart: ^0.4.0
connectivity: ^3.0.6
dio: ^4.0.6
```

### Step 1: Wrap your root application with ProviderScope

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1688223299391/81f200a0-c6e1-4ab5-8127-86808760fe97.png align="center")

### Step 2: Service file for APIsAPI's

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1688223339602/d8beacdf-5354-49ed-b3bf-b060bbb757f8.png align="center")

Here we have created three functions for each of the APIs using the Dio package. For error handling, we are using fpdart package. For that, we have created a typedef to avoid big syntax -

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1688223387729/bf07cd86-493a-49ce-b0cf-09a5d4457dae.png align="center")

Either is a keyword from the fpdart package that we imported. It allows us to return different data types from a single function. In our case, the getSongs() function will either return an HTTP Response or an error string.

### Step 3: Model classes for Song and lyrics

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1688223426312/60435094-5863-47b4-baea-33dfaf6f2263.png align="center")

### Step 4: Home Page Controller

Let’s build the first page where we will list down all the songs. So create a home\_controller.dart file where we are going to create our first provider -

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1688223484138/631b08ea-e249-4e2e-8bb2-79dbeafa593b.png align="center")

Let’s first discuss the controller and then we will cover the provider. In the HomeController class, we have an instance of our SongService class that we created and we have the loadSongs() function which on success stores the songs in the List but an error throws an Exception.

First, we created a provider for HomeController so that by using ref we can use HomeController in the application from anywhere. 

Now we have created a FutureProvider because we are dealing with a network and we need to handle all the loading, loaded and error cases. So for such situations, FutureProvider is best suited. Now we have business logic sorted out, let’s connect it with UI.

### Step 4: Home Page UI

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1688223591528/2bb53c02-295d-44ec-8947-e30ba0e38989.png align="center")

I have created separate widgets for the error page and loading screen, you can check out the GitHub repository for that.

### Step 5: Song & Lyrics Page Controller

Now with the same approach let’s create our Lyrics screen but in this, we need to do two API calls on one single page.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1688223724363/37fb657e-a541-4524-85a0-4944fa5b6ec8.png align="center")

We have again done the same thing with controllers and providers but in these FutureProviders we have taken the input of id which is required for the API.

### Step 6: Song Details Page UI

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1688223968228/5009d42b-0e98-4626-b1dd-df9d927fb94a.png align="center")

> Note: Make sure to wrap your widget with ConsumerWidget to update the UI when the provider updated its data.

### Step 7: Connectivity Provider

We have completed the features of the application but we want to add a simple connectivity check to show the user that he/she is not connected to the internet. So we create a connectivity controller - 

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1688224024135/9e7a6ee2-e7ad-41d1-8924-3358665b2cd5.png align="center")

This class is extended with a StateNotifier of type boolean as we want to only use one value from the class. So for creating a provider for this class we also need to create a StateNotifierProvider. This class just listens to the connectivity stream and updates its state value when there is a change in connection. We use this provider with UI like this -

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1688224086509/e5dbbe3d-b94b-42fa-b9d8-69d9d66a1341.png align="center")

And with that, we have completed our Flutter Music Application using Riverpod. 

**Congratulations** on completing our blog and Flutter music app project with Riverpod! Subscribe to our newsletter for upcoming blogs.

  
**Keep Fluttering** 💙💙💙