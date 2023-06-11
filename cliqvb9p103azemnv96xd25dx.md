---
title: "Supercharge Your Flutter App with Hive: Next-Level Caching for Lightning-Fast Performance"
datePublished: Sun Jun 11 2023 03:30:42 GMT+0000 (Coordinated Universal Time)
cuid: cliqvb9p103azemnv96xd25dx
slug: flutter-hive
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1686370704644/72247fc9-e195-47f1-a93a-280ec53934b7.png
tags: app-development, flutter, caching, hive, wemakedevs

---

### **Introduction:-** 

In today's fast-paced digital world, where users demand seamless experiences and instant access to information, efficient data caching has become a crucial aspect of mobile app development. To enhance the user experience and optimize the performance of Flutter apps, developers often turn to powerful caching solutions. One such solution is the Flutter Hive package, which offers a lightweight and efficient way to store and retrieve data locally. 

In this blog, we will explore the ins and outs of the Flutter Hive package and delve into the benefits it brings to the table. We will learn how to leverage Hive to store and manage data efficiently, enabling faster data retrieval and reducing reliance on remote servers. We will also take a look at the limitations of the Hive package. By the end, you will have a comprehensive understanding of how Hive can supercharge your Flutter app's caching capabilities, leading to improved performance and delighted users.

### **What is Hive?**

Hive is a lightweight and efficient local data storage solution for Flutter applications. It is designed to provide a fast, persistent, and easy-to-use key-value store, making it an excellent choice for caching data and reducing reliance on remote servers. At its core, Hive utilizes the power of NoSQL databases and stores data in a binary format, resulting in blazing-fast read-and-write operations. Another significant advantage of Hive is its support for data encryption. It provides options to encrypt data stored in Hive boxes, adding an extra layer of security to sensitive information. Hive follows a schema-less approach, meaning developers are not required to define rigid data models or schemas beforehand. In the following sections of this blog, we will explore Hive's installation and setup process, dive into its core concepts, and demonstrate practical examples of how to leverage Hive effectively in Flutter applications.

### **Installation and Setup:-** 

To install Hive we need to set up some libraries in pubspec.yaml file - 

```yaml
dependencies:
     hive: ^2.2.3
 	 hive_flutter: ^1.1.0
dev_dependencies:
	 hive_generator: ^2.0.0
  	 build_runner: ^2.3.3
```

After that go to the main.dart file and add this before the run app method.

```dart
WidgetsFlutterBinding.ensureInitialized();
await Hive.initFlutter();
```

With this, we have completed the set-up of Hive.

### **Caching for Primitive Data types:-** 

In Hive, a "Box" is a fundamental concept that represents a named collection of key-value pairs. It acts as a container or storage unit for storing and retrieving data within the Hive database. Think of a Box as a virtual box that holds related data items. Each Box in Hive is identified by a unique name and can be thought of as a separate storage compartment within the overall Hive database. So we need to declare these boxes before we run our application, that means before the run app method.

So add the following code before run app method but after the Hive init method.

```dart
await Hive.openBox(‘Demo’);
```

We have given a box a unique name i.e. Demo in our case, you can name it anything. Boxes in Hive are persistent, which means the data stored in a Box remains available even if the application is closed or the device is restarted. The data is stored in a file on the device's storage, making it accessible across app sessions.

Now let’s perform CRUD operations using this Hive Box.

1. Create -
    
    We are going to just store some string values inside our Hive box with the key as userId. The key is important because we are going to use it to access the values from the local database.
    
    ```dart
    var box = await Hive.openBox('Demo');
    box.put('userId', 'asdki4564adfd');
    ```
    
    With this, we can store key-value pairs of data in our Hive database.
    
2. Read -
    
    We are going to access the stored userId and going to print it so for that use -
    
    ```dart
    var box = await Hive.openBox('Demo');
    print('USERID: ${box.get('userId')}');
    ```
    
3. Update -
    
    Now we want to update the user id stored in the Hive database so that the same syntax is used as of Create.
    
    ```dart
    var box = await Hive.openBox('Demo');
    box.put('userId', '45465asdfeds2d4');
    print('USERID: ${box.get('userId')}');
    ```
    
4. Delete -
    
    If we want to delete any particular key-value pair from the database we use -
    
    ```dart
    var box = await Hive.openBox('Demo');
    box.delete(‘userId’);
    ```
    

All these CRUD operations are very easy for primitive data types like int, strings, booleans, etc. But if we want to store our custom data types in local storage then some extra stuff is required.  

---

### **Caching for User-Defined Models:-**

The first step to adding User Defined Models for caching is to create a Model class. So I am going to use one of my projects which follows [Clean Architecture For Flutter](https://github.com/PranavMasekar/Music-App/tree/Clean-Architecture) in which we built a music application. So we are going to build caching functionality to store all songs from the API. So I have already created the basic model but that is not going to work for caching.

```dart
import 'package:equatable/equatable.dart';

class Song extends Equatable {
  final int songId;
  final String songName;
  final int albumId;
  final String albumName;
  final String artistName;

  Song({
    required this.songId,
    required this.songName,
    required this.albumId,
    required this.albumName,
    required this.artistName,
  });

  @override
  List<Object?> get props => [songId, songName, albumId, albumName, artistName];
}
```

We need to change our Model structure to this - 

```dart
import 'package:equatable/equatable.dart';
import 'package:hive/hive.dart';
part 'song_entity.g.dart';

@HiveType(typeId: 0)
class Song extends Equatable {
  @HiveField(0)
  final int songId;
  @HiveField(1)
  final String songName;
  @HiveField(2)
  final int albumId;
  @HiveField(3)
  final String albumName;
  @HiveField(4)
  final String artistName;

  Song({
    required this.songId,
    required this.songName,
    required this.albumId,
    required this.albumName,
    required this.artistName,
  });

  @override
  List<Object?> get props => [songId, songName, albumId, albumName, artistName];
}
```

* First, we have annotated our class with HiveType and given it a unique id. This is necessary because we are going to use build runner to create Adapter automatically. 
    
* Adapters are required to convert the object from and to binary form.
    
* After that, each variable in the class is annotated with HiveField and we are given some unique value.
    
* Then run the following command to generate the &lt;filename&gt;.g.dart file
    
    ```bash
    flutter packages pub run build_runner build
    ```
    
* Head towards the main.dart and add this line just after the Hive init method.
    
    ```dart
    Hive.registerAdapter(SongAdapter());
    await Hive.openBox<Song>('songs');
    ```
    
* Here we have created a new box named Songs and given its type Song.
    

Now let’s do some CRUD operations on our custom model - 

```dart
@override
  Future<Either<Failure, List<Song>>> getSongs() async {
    if (await networkInfo.isConnected) {
      try {
        final songs = await remoteDataSource.getSongs();
        for (SongModel song in songs) {
          box.put(song.songId, song);
        }
        log("Returned from Remote Datasource");
        return right(songs);
      } on ServerException {
        return left(ServerFailure());
      }
    } else {
      List<Song> songs = box.values.cast<Song>().toList();
      log("Returned from Hive Datasource");
      return right(songs);
    }
  }
```

* We have a box.put() method to store a single class object in local storage and we have given the id the same as the song id.
    
* To get a single value from the box we use box.get() function but to get the entire content of the box we use box.values and then we cast it to our correct type i.e. Song.
    
* Delete operation is same as of primitive data types we use box.delete(‘key’);
    

### **Limitations of Hive:-** 

1. As we see, we need to provide some unique IDs to our custom classes to work with Hive. But if we change the unique identifiers then it can cause some inconsistencies. 
    
2. Also, variables have identifiers so suppose we have a variable named userId of type String and with ID 1 and we removed this variable and added a new variable named phoneNumber of type int and we gave the same ID as 1, Then this is going to cause serious problems on already installed applications.
    
3. Hive supports indexing to improve query performance. However, it has limitations on the types of fields that can be indexed. Only integer, string, and DateTime fields can be indexed, limiting the flexibility for optimizing queries based on other data types.
    

**Congratulations!!!** You have taken a significant step towards optimizing your Flutter app's performance and user experience by exploring the capabilities of Flutter Hive. While it's important to consider Hive's limitations such as the lack of relational database features and limited querying options. 

  
**Happy Fluttering!!!** 💙💙💙