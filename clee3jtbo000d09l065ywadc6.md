# Flutter Notifications

One of the essential features of any mobile app is the ability to send notifications to users. Notifications can be used to alert users about new content, remind them of important events, or simply keep them engaged with the app. In this blog, we will explore how to implement notifications in Flutter apps. Whether you are a seasoned Flutter developer or just getting started, this blog will provide you with the knowledge and tools you need to implement notifications in your app and enhance your users' experience.

Prerequisites for this article - 

* Flutter application with Firebase connection
    
* Firebase Project 
    

For the implementation of notifications in flutter we need to import some packages i.e. 

* flutter\_local\_notifications: ^12.0.4
    
* firebase\_messaging: ^14.2.4
    

So before going to the actual implementation let’s first configure our firebase project for app notifications.

For that open the Firebase console and navigate to All Products and then to Cloud Messaging.

Click on Create your first campaign

Select Firebase Notification messages

Now enter a sample notification details like title , text , etc and click on next.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1676973715989/2badda38-292c-43a3-be39-c0859979f821.png align="center")

In Target select your application and click next.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1676973752148/fb0b9177-7718-4942-83fc-998ff3420797.png align="center")

Let other fields be the default and click next on every other step. Click on Save as draft.

That’s it we have configured our Firebase application for Notifications.

Before going to actual coding let’s first revise the concept of notifications in the application. There are two types of notifications i.e. Background Notifications and Foreground Notifications.

Background Notifications are notifications that are received when the application is in the dead state means the application is not open.

Foreground Notifications are notifications that are received when the application is in the active state means the application is running or in the background.

By default background, notifications are handled by firebase which means we don’t need to handle the manual show notification. But for the foreground notification firebase by default doesn’t show the notification so we need to create a notification on our own, which we are going to do with the help of the Local notifications package.

Now let’s do some coding 💪

We are going to create two services, one local notification service will be responsible for creating foreground notifications. The second will be to handle firebase background notifications.

So let’s first create a firebase notification service, create a file named notification service

```dart
import 'dart:developer';
import 'package:firebase_messaging/firebase_messaging.dart';
import 'local_notification_service.dart';

class FirebaseMessagingMethods {
  final FirebaseMessaging _firebaseMessaging = FirebaseMessaging.instance;

  Future<String?> get token async => await _firebaseMessaging.getToken();

  void listenOnMessage() async {
    // Notification Stack
    _firebaseMessaging.getInitialMessage().then((message) {
      if (message != null) {
        log(message.toMap().toString());
      }
    });
    // Listen Foreground notification
    FirebaseMessaging.onMessage.listen((RemoteMessage? message) {
      if (message != null) {
        log(message.notification!.title!);
        log(message.notification!.body!);
        // Show local notification
        LocalNotificationServices.display(message);
      }
    });
  }

  // Background Notification OnTap
  void listenOnMessagedAppOpened() {
    FirebaseMessaging.onMessageOpenedApp.listen((RemoteMessage message) {
      log(message.data['data']);
    });
  }
}
```

* First, we create an instance of Firebase Messaging
    
* The listenOnMessage function is used to read the notification which occurred while the application was in a dead state.
    
* We are using a display method from our Local Notification service to show the notification.
    
* The listenOnMessagedAppOpened function is used to handle the onTap of background notification.
    

Now let’s create a local notification service

```dart
import 'package:firebase_messaging/firebase_messaging.dart';
import 'package:flutter_local_notifications/flutter_local_notifications.dart';

class LocalNotificationServices {
  static final FlutterLocalNotificationsPlugin _notificationsPlugin =
      FlutterLocalNotificationsPlugin();

  static onDidReceiveLocalNotification(
      int id, String? title, String? body, String? payload) async {
    // Foreground Notification OnTap
  }

  static void initialize() {
    final InitializationSettings initializationSettings =
        InitializationSettings(
            android: AndroidInitializationSettings('@mipmap/ic_launcher'),
            iOS: DarwinInitializationSettings(
                onDidReceiveLocalNotification: onDidReceiveLocalNotification));
    _notificationsPlugin.initialize(
      initializationSettings,
      onDidReceiveNotificationResponse: (details) {},
    );
  }

  static void display(RemoteMessage message) async {
    try {
      final id = DateTime.now().millisecondsSinceEpoch ~/ 1000;

      final NotificationDetails notificationDetails = NotificationDetails(
        android: AndroidNotificationDetails(
          'nscc',
          'nscc channel',
          channelDescription: 'Channel for nscc to get Notifications',
          importance: Importance.high,
          priority: Priority.high,
        ),
        iOS: DarwinNotificationDetails(),
      );
      await _notificationsPlugin.show(id, message.notification!.title,
          message.notification!.body, notificationDetails);
    } catch (e) {
      rethrow;
    }
  }
}
```

* First, we need to define initial settings for notifications so we create an initialize method we define android and IOS settings.
    
* Then we create a display method to display the notification, in this method we create an object of NotificationDetails.
    
* The onDidReceiveLocalNotification function is used to handle the onTap of foreground notification.
    

Now let’s go to main.dart to call these methods -

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(options: DefaultFirebaseOptions.currentPlatform);
  LocalNotificationServices.initialize();
  FirebaseMessaging.onBackgroundMessage(backGroundMessageHandler);
  runApp(const MyApp());
}
```

```dart
class MyApp extends StatefulWidget {
  const MyApp({super.key});

  @override
  State<MyApp> createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  @override
  void initState() {
    super.initState();
    FirebaseMessagingMethods().listenOnMessage();
  }

  @override
  Widget build(BuildContext context) {
    return ScreenUtilInit(
      designSize: const Size(360, 840),
      minTextAdapt: true,
      builder: (context, child) {
        return GetMaterialApp(
          debugShowCheckedModeBanner: false,
          initialRoute: RoutesNames.authPage,
          getPages: AppRoutes.routes,
          theme: ThemeData(
            fontFamily: "Poppins",
          ),
        );
      },
    );
  }
}
```

Before the run app method, we need to initialize the notification services. In MyApp class we call LocalNotification.listenOnMessage() method to listen to foreground messages.

So all the preparations are almost done. The only thing remaining is to configure Local Notification Plugin for the flutter application which can be done by official documentation of the plugin [LINK](https://pub.dev/packages/flutter_local_notifications#-android-setup).

Now run the application, after successful execution of the application the token will be printed on the debug console, copy that token.

To trigger notifications you can use the firebase console or postmen.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1676974288815/49f8503e-54ac-4476-a4b3-87f4b8a7ae8a.png align="center")

Click on publish.

Now to trigger notification through postmen we need to have a server key, which can be obtained from the firebase console.

Go to project settings and then Cloud Messaging. Then enable Cloud Messaging API. This will redirect you to GCP Console there just click on enable.

After doing this you will see a screen like this 

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1676974336463/cd76c688-5b5a-48dc-b208-ba76ba0ea733.png align="center")

Now head on to Postmen and create a new POST request -

URL :- [https://fcm.googleapis.com/fcm/send](https://fcm.googleapis.com/fcm/send)

Headers : 

* Authorization : key=&lt;ServerKey&gt;
    
* Content-Type : application/json
    

Body :

```json
{
    "to":"<TOKEN>",
    "notification":{
        "body":
            "New Event is coming",
        "title": "New Event is coming",
        "sound":"tone.aiff"
    },

    "data": {
        "title": "",
        "data": "Demo notification",
        "type": " " 
      }
}
```

Now send the request and a notification will be triggered.

Congratulations !! 🎉 🎉 You have successfully integrated notifications functionality in the Flutter application.