---
title: "Dynamic App Configuration with Flutter and Firebase"
datePublished: Sun Jul 30 2023 03:30:13 GMT+0000 (Coordinated Universal Time)
cuid: clkovvdnb000309mr69yehoeo
slug: remote-config
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1690649052023/f227c390-5485-499b-bfbc-edc60977ef9d.png
tags: flutter, wemakedevs, riverpod, flutterfire, remoteconfig

---

# Introduction -

Flutter is an increasingly popular open-source framework for building beautiful, high-performance mobile apps for iOS and Android from a single codebase. One of its major advantages is the ability to iterate quickly and ship updates on the fly. However, sometimes you may want to tweak app behaviour or make configuration changes without needing to push full app updates. This is where a tool like Firebase Remote Config comes in handy!

Firebase Remote Config allows you to dynamically change app behaviour and appearance remotely by storing your configuration parameters on Firebase servers. Any changes can be pushed immediately without publishing a new app version. In this post, we'll explore how to integrate Firebase Remote Config into a Flutter app to unlock powerful remote configuration capabilities. I'll cover creating and publishing parameter values, and retrieving those values within your Dart code to update UI elements, business logic, feature flags, and more. Let's dive in!

### **Prerequisites :** 

* Flutter Project with Firebase setup
    
* Riverpod Knowledge
    
* Excitement to update application overfly
    

# **Firebase Remote Config Setup -**

Before actually starting the coding part we need to set up some things on Firebase Console so open the Remote config page on Firebase console.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690647346492/06efed3d-ad9c-42f8-8671-fe4049aef927.png align="center")

After that create configuration and enter the data which we want to receive in the application. For the start, we are going through a simple example of changing the colour of the container. So enter the data like this  - 

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690647377952/69473893-2e2f-4092-97a1-178be011584f.png align="center")

After that hit save and then publish your changes.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690647401919/d197e206-f9ff-4e5b-9c4b-6ecce7d848af.png align="center")

Okay with that we have set up the remote config for the project. Now let’s handle some application-level logic.

# **Flutter Set up -**

First, we need to import some packages to work with Firebase remote config -

```yaml
  firebase_core: ^2.15.0
  flutter_riverpod: ^2.3.6
  firebase_remote_config: ^4.2.4
```

Make sure to add these lines in the main.dart file after installing packages -

```dart
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  runApp(const ProviderScope(child: MyApp()));
}
```

Now let’s create our remote config service file which will connect with Firebase and retrieve the data and send this data to UI via Riverpod Provider.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690647510919/30e508d1-2deb-4957-917a-972bfe6d8eaa.png align="center")

We create an init method to initialize our remote config. We set some properties of the config like timeout duration and minimum fetch interval. In the end, we use the fetchAndActivate() method. This fetchAndActivate() method in Firebase Remote Config is used to fetch the latest parameter values from the Firebase server and activate them for use in your app.

Then we create a simple getter to get colour from the remoteConfig variable.

> Note: Make sure that the key “Color” is matching with the variable name that we did in the previous step.

At the start, we created Notifier Provider which holds the RemoteConfigService class and a bool value. We also created a colorProvider which just provides the colour code to the UI.

So let’s take a look at the UI file - 

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690647595741/0e730392-3c94-4f72-8342-3b02e134939a.png align="center")

Now run the application you will see the container with the provided colour code in this case it's orange colour. Now change the value of the variable from the Firebase console and restart the app. You will see the container with the updated colour from the firebase. 

Great!!! But this is just a small example of how powerful Firebase remote config can become. One of the most popular use cases of Firebase remote config is to force users to update the application. So let’s take a look into this scenario.

# **Force App Update -**

Just like before doing anything with the code let’s set up the Firebase remote config with variables.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690647653311/36152c75-ce03-4a60-84e2-b7d89aa35346.png align="center")

And hit save and publish the changes. First change the version in pubspec.yaml file to 1.0.4 for testing. And add this package to your flutter project - 

```yaml
package_info_plus: ^4.0.2
```

We use this package to get the current version of the application. Now let’s create a getter for this in our service file.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690647721503/ef7e5e27-ccfa-4454-8556-fb9794117939.png align="center")

Here we use the ref.listen method to listen to the changes of the configProvider. In our case, we have a condition if the next is false then check for the update. Here next is false means our remote config initialization is complete and we can access the data. Inside the checkForUpdate method, we just check if the current application version is less than the latest version and if yes then is it a force update. Accordingly, we show the update dialogue box to the user. 

We can now bump up the application version from the application and rerun the application to see whether a force update is triggered or not. 

All of the code is in this [repository](https://github.com/PranavMasekar/custom_painter/tree/remote-config)

**Congratulations**, you've reached the end of this blog! By now, you should have a solid understanding of how to configure, fetch, and activate remote parameter values in your Dart code. You also got hands-on experience applying Remote Config in practice with examples like dynamically changing the theme color and forcing application updates.

As you continue on your Flutter development journey, remember the flexibility that solutions like Remote Config provide. No need to go through the entire build and release process just to tweak some configuration values. You can now release updates on the fly and provide a more dynamic experience for users.

Keep Fluttering 💙💙💙