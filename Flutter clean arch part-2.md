---
title: "Unlocking the Secrets of Clean Architecture: Building Scalable Flutter Apps : Part-2"
datePublished: Tue May 23 2023 03:30:42 GMT+0000 (Coordinated Universal Time)
cuid: clhzpy31r0bvmrrnv2e1p265o
slug: flutter-clean-architecture-part-2
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1684564734444/fb3c87ba-27f3-435e-b2fd-cfef4cd275d6.png
tags: flutter, dependency-injection, clean-architecture, flutter-bloc, wemakedevs

---

In our previous part of Clean Architecture with Flutter, we learned about Clean Architecture concepts and layers. We also had preparation for our music app in Flutter. We installed the required packages and prepared our folder structure. If you haven’t gone through part-1 I highly recommend doing so.

In this part, we are going to implement the different layers one by one. We are going to start with the domain layer then the data layer and at the end presentation layer.

🧑‍💻 **Let’s start with Domain Layer -**

As explained in the previous part, the domain layer consists of entities, repository contracts and useCases. So first we are going to create entities. So create two files inside the entities folder - 

1. song\_entity.dart
    
2. lyrics\_entity.dart
    

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

```dart
import 'package:equatable/equatable.dart';

class Lyrics extends Equatable {
  final int lyricsId;
  final String lyrics;

  Lyrics({
    required this.lyrics,
    required this.lyricsId,
  });

  @override
  List<Object?> get props => [lyrics, lyricsId];
}
```

These entities are the models which are going to be used in the UI, so accordingly, only required parameters are selected as class members. Now let’s create repository contracts for the useCases. So create song\_repository.dart in the repositories section.

```dart
import 'package:fpdart/fpdart.dart';
import 'package:music_app/core/error/failure.dart';
import 'package:music_app/features/songs/domain/entities/lyrics_entity.dart';
import 'package:music_app/features/songs/domain/entities/song_entity.dart';

abstract class SongRepository {
  Future<Either<Failure, List<Song>>> getSongs();
  Future<Either<Failure, Lyrics>> getLyrics(int id);
  Future<Either<Failure, Song>> getSong(int id);
}
```

Repository contracts are just abstract class that defines the functions that must be implemented by the domain repository class. We list down the methods that are required in the application like getSongs(), getLyrics(), and getSong().In repository contracts, we just define our function their actual implementation will be in domain-layer repositories.

But what is this Either 🤔 ??

Either is a keyword from the fpdart package that we imported. It allows us to return different data types from a single function. In our case, the getSongs() function will either return a list of songs or an instance of the Failure class.

In Either keyword generally, the left part represents error and the right part represents our data.

 Failure is the custom class that we had to define, this class will be used throughout the application so we need to define it inside the core. So go to the error folder and create a failure.dart file -

```dart
import 'package:equatable/equatable.dart';

abstract class Failure extends Equatable {
  final List properties;
  const Failure({this.properties = const []});

  @override
  List<Object?> get props => properties;
}

class ServerFailure extends Failure {
  const ServerFailure();
}

class CacheFailure extends Failure {
  const CacheFailure();
}

class NoInternetConnectionFailure extends Failure {
  const NoInternetConnectionFailure();
}
```

Failure class is also an abstract class and we need to create custom Failures according to our application needs. In our application we are going to use two types of failures - 

1. ServerFailure - Failure from API Server
    
2. NoInternetConnectionFailure - Failure due to internet connection
    

Our application is going to have three useCases - 

1. Getting all songs
    
2. Getting one song
    
3. Getting the lyrics of a song
    

So accordingly we create the useCase of the application so create get\_songs.dart, get\_song.dart, get\_lyric.dart.

Before actually coding the useCases first let’s create a base class for useCase which will define the behavior of the useCase and will be used by every useCase in the application. So under the core/useCases folder create usecase.dart.

```dart
import 'package:equatable/equatable.dart';
import 'package:fpdart/fpdart.dart';
import 'package:music_app/core/error/failure.dart';

abstract class UseCase<Type, Params> {
  Future<Either<Failure, Type>> call(Params params);
}

class NoParams extends Equatable {
  @override
  List<Object?> get props => [];
}
```

Again base class is an abstract class that has only one method call. But before going to the call method what’s that  &lt;Type, Params&gt;?

We are using generics of dart to define a return type of the function. Type variables represent the data that will be returned when the function is successfully executed. Params is the class that defines the parameters that the method uses. Each useCase will have its own params class because each method requires a different set of parameters. The function return type is similar to what we have in repository contracts. Instead of giving concrete type to the right side, we have to use Generics. We have also defined the NoParams class which is self-explanatory i.e. if a function doesn’t require any parameters then we will use this Noparam Class.

So let’s start coding actual useCase classes - 

**GetSongsUseCase -** 

```dart
import 'package:fpdart/fpdart.dart';
import 'package:music_app/core/error/failure.dart';
import 'package:music_app/core/usecases/usecase.dart';
import 'package:music_app/features/songs/domain/entities/song_entity.dart';
import 'package:music_app/features/songs/domain/repositories/song_repository.dart';

class GetSongsUseCase extends UseCase<List<Song>, NoParams> {
  final SongRepository songRepository;

  GetSongsUseCase({required this.songRepository});

  @override
  Future<Either<Failure, List<Song>>> call(NoParams params) async {
    return await songRepository.getSongs();
  }
}
```

As said earlier each useCase class extends the base class. Notice here we have defined the concrete type of UseCase base class i.e. List&lt;Song&gt;. Then we take the instance of the song repository class as input in the constructor. Now as it extends the base class we need to implement the call() method.

Pay attention here - 

`Future<Either<Failure, Type>> call(Params params);`

This is the structure of the function in the base useCase class.

`Future<Either<Failure, List<Song>>> call(NoParams params){}`

This is the structure of the function in the actual implementation of useCase see how just Type is being replaced by List&lt;Song&gt; everywhere in the function.

In the implementation, we just return the response from the getSongs() function from the repository.

**Is there any special reason for the naming method as call ??**

Yes, it's a special method. Every class in Dart has this callable function named call(). This method is useful because we don't need to separately call this function.

Let’s take the GetSongUseCase example itself, suppose instead of call we named the function as myFunction.

So to use the function we will do something like this -

```dart
GetSongUseCase object = GetSongUseCase();

result  = object.myFunction();
```

But using the call method we can simplify this as - 

```dart
result = GetSongUseCase(); 
```

The call method will automatically get executed for us.

**GetLyricsUseCase -**

All the other useCases have very similar implementations besides the return type of function but I wanted to explain the custom Params class. 

```dart
import 'package:equatable/equatable.dart';
import 'package:fpdart/fpdart.dart';
import 'package:music_app/core/error/failure.dart';
import 'package:music_app/core/usecases/usecase.dart';
import 'package:music_app/features/songs/domain/entities/lyrics_entity.dart';
import 'package:music_app/features/songs/domain/repositories/song_repository.dart';

class GetLyricsUseCase extends UseCase<Lyrics, Params> {
  final SongRepository songRepository;

  GetLyricsUseCase({required this.songRepository});

  @override
  Future<Either<Failure, Lyrics>> call(Params params) async {
    return await songRepository.getLyrics(params.id);
  }
}

class Params extends Equatable {
  final int id;
  Params({required this.id});

  @override
  List<Object?> get props => [id];
}
```

GetLyricsUseCase requires a parameter of SongId to get the appropriate lyrics. So we create a class that takes the input of ID. Similarly, we can create any custom Params classes.

> **Note:** Name these params classes carefully because if you create every class with a Params name it will cause very much trouble when you import multiple files containing the same params class.

**GetSongUseCase -** 

```dart
import 'package:equatable/equatable.dart';
import 'package:fpdart/fpdart.dart';
import 'package:music_app/core/error/failure.dart';
import 'package:music_app/core/usecases/usecase.dart';
import 'package:music_app/features/songs/domain/entities/song_entity.dart';
import 'package:music_app/features/songs/domain/repositories/song_repository.dart';

class GetSongUseCase extends UseCase<Song, SongParams> {
  final SongRepository songRepository;

  GetSongUseCase({required this.songRepository});

  @override
  Future<Either<Failure, Song>> call(SongParams params) async {
    return await songRepository.getSong(params.id);
  }
}

class SongParams extends Equatable {
  final int id;
  SongParams({required this.id});

  @override
  List<Object?> get props => [id];
}
```

Similar to GetLyricsUseCase the GetSongUseCase is implemented only difference is having a different return type and Params class.

With this, we have completed our first layer i.e. our domain layer. Next, we are going to code the data layer.

---

**🧑‍💻 Let’s start with Data Layer -** 

In the data layer first, we will create models then repositories and data sources. So let’s first create the models which will also contain json conversion logic.

Create lyrics\_model.dart and song\_model.dart.

```dart
import 'package:music_app/features/songs/domain/entities/song_entity.dart';

class SongModel extends Song {
  final int artistId;
  final int songRating;
  SongModel({
    required super.songId,
    required super.songName,
    required super.albumId,
    required super.albumName,
    required super.artistName,
    required this.artistId,
    required this.songRating,
  });

  factory SongModel.fromJson(Map<String, dynamic> json) {
    return SongModel(
      songId: json["track_id"],
      songName: json["track_name"],
      albumId: json["album_id"],
      albumName: json["album_name"],
      artistName: json["artist_name"],
      artistId: json["artist_id"],
      songRating: json["track_rating"],
    );
  }

  Map<String, dynamic> toJson() {
    return {
      "track_id": songId,
      "track_name": songName,
      "album_id": albumId,
      "album_name": albumName,
      "artist_id": artistId,
      "artist_name": artistName,
      "track_rating": songRating,
    };
  }
}
```

```dart
import 'package:music_app/features/songs/domain/entities/lyrics_entity.dart';

class LyricsModel extends Lyrics {
  final String copyright;
  LyricsModel({
    required super.lyrics,
    required super.lyricsId,
    required this.copyright,
  });

  factory LyricsModel.fromJson(Map<String, dynamic> json) {
    return LyricsModel(
      lyrics: json["lyrics_body"],
      lyricsId: json["lyrics_id"],
      copyright: json["lyrics_copyright"],
    );
  }

  Map<String, dynamic> toJson() {
    return {
      "lyrics_body": lyrics,
      "lyrics_id": lyricsId,
      "lyrics_copyright": copyright,
    };
  }
}
```

This is pretty standard code that we use in every API calling in Flutter for converting json into a dart object and vice-versa.

So let’s first create an abstract class for data sources, so inside the data sources folder create song\_remote\_datasource.dart.

```dart
import 'dart:convert';
import 'package:dio/dio.dart';
import 'package:music_app/core/error/exceptions.dart';
import 'package:music_app/features/songs/data/models/lyrics_model.dart';
import 'package:music_app/features/songs/data/models/song_model.dart';

abstract class SongRemoteDataSource {
  Future<LyricsModel> getLyrics(int id);
  Future<List<SongModel>> getSongs();
  Future<SongModel> getSong(int id);
}

class SongRemoteDataSourceImplementation extends SongRemoteDataSource {
  Dio dio = Dio();

  @override
  Future<LyricsModel> getLyrics(int id) async {
    String url =
        "https://api.musixmatch.com/ws/1.1/track.lyrics.get?track_id=$id&apikey=<YOUR_API_KEY>";

    final response = await dio.get(url);

    if (response.statusCode == 200) {
      Map<String, dynamic> res = jsonDecode(response.data);
      return LyricsModel.fromJson(res["message"]["body"]["lyrics"]);
    } else {
      throw ServerException();
    }
  }

  @override
  Future<List<SongModel>> getSongs() async {
    String url =
        "https://api.musixmatch.com/ws/1.1/chart.tracks.get?apikey=<YOUR_API_KEY>";

    final response = await dio.get(url);

    if (response.statusCode == 200) {
      Map<String, dynamic> res = jsonDecode(response.data);

      List<SongModel> songs = [];
      for (var element in res["message"]["body"]["track_list"]) {
        songs.add(SongModel.fromJson(element["track"]));
      }
      return songs;
    } else {
      throw ServerException();
    }
  }

  @override
  Future<SongModel> getSong(int id) async {
    String url =
        "https://api.musixmatch.com/ws/1.1/track.get?track_id=$id&apikey=<YOUR_API_KEY>";

    final response = await dio.get(url);

    if (response.statusCode == 200) {
      Map<String, dynamic> res = jsonDecode(response.data);
      return SongModel.fromJson(res["message"]["body"]["track"]);
    } else {
      throw ServerException();
    }
  }
}
```

Abstract class contains definitions of required functions i.e. getSongs(), getSong(), getLyrics(). An important point of these functions is their return types. All these functions have concrete return types and no return types with Either format. 

Here is the point to remember while coding the data layer. 

DataSources will generate Exceptions. 

Repositories will handle these Exceptions and will return Failure Object.

After the abstract class, we implement our APIs with the Dio package. Observe that we have a check on the status code. If the status code is 200 then we are returning the appropriate data else we are returning ServerException(). These are the custom Exceptions that we have to create. So in core/error create exceptions.dart file

```dart
class ServerException implements Exception {}

class CacheException implements Exception {}

class NoInternetException implements Exception {}
```

We have just created some exceptions extending the baseline exception class from Dart.

Make sure to use your API Keys in the URL. You can get it from Musixmatch's official documentation page. With this, we also completed a data sources file. Now let’s code the repository file, so create a song\_repository\_implementation.dart file inside the repository section.

```dart
import 'package:fpdart/fpdart.dart';
import 'package:music_app/core/error/exceptions.dart';
import 'package:music_app/core/error/failure.dart';
import 'package:music_app/core/network/network_info.dart';
import 'package:music_app/features/songs/data/datasources/song_remote_datasource.dart';
import 'package:music_app/features/songs/domain/entities/lyrics_entity.dart';
import 'package:music_app/features/songs/domain/entities/song_entity.dart';
import 'package:music_app/features/songs/domain/repositories/song_repository.dart';

class SongRepositoryImplementation extends SongRepository {
  final SongRemoteDataSource remoteDataSource;
  final NetworkInfo networkInfo;

  SongRepositoryImplementation({
    required this.remoteDataSource,
    required this.networkInfo,
  });

  @override
  Future<Either<Failure, Lyrics>> getLyrics(int id) async {
    if (await networkInfo.isConnected) {
      try {
        final lyrics = await remoteDataSource.getLyrics(id);
        return right(lyrics);
      } on ServerException {
        return left(ServerFailure());
      }
    } else {
      return left(NoInternetConnectionFailure());
    }
  }

  @override
  Future<Either<Failure, List<Song>>> getSongs() async {
    if (await networkInfo.isConnected) {
      try {
        final songs = await remoteDataSource.getSongs();
        return right(songs);
      } on ServerException {
        return left(ServerFailure());
      }
    } else {
      return left(NoInternetConnectionFailure());
    }
  }

  @override
  Future<Either<Failure, Song>> getSong(int id) async {
    if (await networkInfo.isConnected) {
      try {
        final song = await remoteDataSource.getSong(id);
        return right(song);
      } on ServerException {
        return left(ServerFailure());
      }
    } else {
      return left(NoInternetConnectionFailure());
    }
  }
}
```

Song Repository Implementation is of the abstract class defined in the domain layer repository folder. So we need to implement all those functions which are defined in the abstract class. 

In this implementation, we also check for the user's internet connection. So to get the current status of the connection we need to create a networkInfo class which will return a boolean about the current connectivity status. 

This class will be used all over the application so we need to define it in core. So in the core/network folder create network\_info.dart.

```dart
import 'package:connectivity_plus/connectivity_plus.dart';

abstract class NetworkInfo {
  Future<bool> get isConnected;
}

class NetworkInfoImpl extends NetworkInfo {
  final Connectivity connectivity;

  NetworkInfoImpl({required this.connectivity});

  @override
  Future<bool> get isConnected async {
    ConnectivityResult result = await connectivity.checkConnectivity();
    if (result == ConnectivityResult.mobile ||
        result == ConnectivityResult.wifi) {
      return true;
    } else {
      return false;
    }
  }
}
```

We use the connectivity plus package to get the current status of users' internet. The function returns a boolean representing this status only.

Now important points in the implementation of repositories - 

* Syntax of return in Either keyword. As I said Left side represents error and the right side represents data. So if any kind of error is being produced then return left() else return right().
    
* We first check the connectivity status and if the user is not connected to the internet we return NoInternetConnectionFailure().
    
* If a data source generates an Exception we need to handle that exception and return Failure. 
    

And with this file, we have completed our Data layer as well.

Next, we are going to code the presentation layer. 

Note: This blog doesn’t explain the UI Code as it is basic and a prerequisite of this blog post.

---

**🧑‍💻 Let’s start with Presentation Layer -** 

In the presentation layer, we are going to create BLoC and manage the dependencies of the application using the get\_it package. We are not going to code the UI part of the application. If you are not familiar with BLoC please check out my blog series on [BLoC](https://sungod.hashnode.dev/flutter-bloc-beginners-guide-part-1).

So let’s start with creating our BLoC - 

Create song\_bloc.dart , song\_event.dart and song\_state.dart inside the bloc folder.

```dart
part of 'song_bloc.dart';

abstract class SongState extends Equatable {}

class SongsLoadedState extends SongState {
  final List<Song> songs;
  SongsLoadedState({
    required this.songs,
  });
  @override
  List<Object?> get props => [songs];
}

class SongLoadingState extends SongState {
  @override
  List<Object?> get props => [];
}

class SongLoadingFailure extends SongState {
  final String errorMsg;
  SongLoadingFailure({required this.errorMsg});
  @override
  List<Object?> get props => [errorMsg];
}

class SongDetailsLoadedState extends SongState {
  final Song song;
  final Lyrics lyric;

  SongDetailsLoadedState({required this.song, required this.lyric});
  @override
  List<Object?> get props => [song, lyric];
}
```

So we have four states - 

1. SongsLoadedState
    
2. SongLoadingState
    
3. SongLoadingFailure
    
4. SongDetailsLoadedState
    

```dart
part of 'song_bloc.dart';

abstract class SongEvent extends Equatable {}

class GetSongs extends SongEvent {
  @override
  List<Object?> get props => [];
}

class GetDetails extends SongEvent {
  final int id;
  GetDetails({required this.id});

  @override
  List<Object?> get props => [id];
}
```

We have only two events in the application -

1. GetSongs
    
2. GetDetails
    

Now let’s see actual bloc implementation -

```dart
import 'package:bloc/bloc.dart';
import 'package:equatable/equatable.dart';
import 'package:music_app/core/error/failure.dart';
import 'package:music_app/core/repositories/song_storage.dart';
import 'package:music_app/core/usecases/usecase.dart';
import 'package:music_app/features/songs/domain/entities/lyrics_entity.dart';
import 'package:music_app/features/songs/domain/entities/song_entity.dart';
import 'package:music_app/features/songs/domain/usecases/get_lyrics.dart';
import 'package:music_app/features/songs/domain/usecases/get_song.dart';
import 'package:music_app/features/songs/domain/usecases/get_songs.dart';

part 'song_event.dart';
part 'song_state.dart';

class SongBloc extends Bloc<SongEvent, SongState> {
  final GetLyricsUseCase getLyrics;
  final GetSongsUseCase getSongs;
  final GetSongUseCase getSong;
  final SongStorageRepository songStorageRepository;
  SongBloc({
    required this.getLyrics,
    required this.getSongs,
    required this.getSong,
    required this.songStorageRepository,
  }) : super(SongLoadingState()) {
    on<GetSongs>(_getSongs);
    on<GetDetails>(_getDetails);
  }

  String getStringByFailure(Failure failure) {
    if (failure is ServerFailure) {
      return "Server failure";
    } else if (failure is NoInternetConnectionFailure) {
      return "No Internet Connection!!!";
    } else {
      return "Unexpected Error";
    }
  }

  Future<void> _getSongs(GetSongs event, Emitter<SongState> emit) async {
    emit(SongLoadingState());
    final result = await getSongs(NoParams());

    result.fold(
      (failure) => emit(
        SongLoadingFailure(errorMsg: getStringByFailure(failure)),
      ),
      (songs) {
        songStorageRepository.setCurrentSongs(songs);
        emit(SongsLoadedState(songs: songs));
      },
    );
  }

  Future<void> _getDetails(GetDetails event, Emitter<SongState> emit) async {
    emit(SongLoadingState());
    Song? song;
    Lyrics? lyrics;

    final songResult = await getSong.call(SongParams(id: event.id));
    songResult.fold(
      (failure) {
        emit(SongLoadingFailure(errorMsg: getStringByFailure(failure)));
        return;
      },
      (s) => song = s,
    );

    final lyricResult = await getLyrics(Params(id: event.id));
    lyricResult.fold(
      (failure) {
        emit(SongLoadingFailure(errorMsg: getStringByFailure(failure)));
        return;
      },
      (l) => lyrics = l,
    );

    if (song != null && lyrics != null) {
      songStorageRepository.setCurrentLyrics(lyrics!);
      songStorageRepository.setCurrentSong(song!);
      emit(SongDetailsLoadedState(song: song!, lyric: lyrics!));
    } else {
      emit(SongLoadingFailure(errorMsg: "Unexpected Error"));
    }
  }
}
```

To implement bloc we will need all the useCases. In both event handler functions i.e. *getSongs() and* getDetails(), we first emit the Loading State. Then we directly call the useCase. See how we directly use getSongs() and not getSongs.call().

Then useCase will return the Either type response which needs to be handled correctly. So fpdart package provides a fold method to handle the left and right sides of the returned value.

Inside the fold method we will define what happens useCase returns a left value i.e. Failure or a right value i.e. data. 

If it's a failure we return the Error state with proper string value. Else we return the Loaded state with actual data. 

In the \_getDetails() function we need to call two useCase first to get song details and then lyrics. So we handle each useCase response one after the other and store both values in the Song and Lyrics object. 

> Note: The songStorageRepository is not related to clean architecture, it's the BLoC repository where we store our data temporarily. BLoC repositories are very useful to store data. If you want you can also implement a BLoC repository, I have already explained this in my BLoC series but it's not mandatory to have a storage repository.

Now our BLoC is completed, let's consider our dependencies of the application. 

Create a locator.dart file inside the lib folder.

```dart
import 'package:connectivity_plus/connectivity_plus.dart';
import 'package:get_it/get_it.dart';
import 'package:music_app/core/network/network_info.dart';
import 'package:music_app/core/repositories/song_storage.dart';
import 'package:music_app/features/songs/data/datasources/song_remote_datasource.dart';
import 'package:music_app/features/songs/data/repositories/song_repository_impl.dart';
import 'package:music_app/features/songs/domain/repositories/song_repository.dart';
import 'package:music_app/features/songs/domain/usecases/get_lyrics.dart';
import 'package:music_app/features/songs/domain/usecases/get_song.dart';
import 'package:music_app/features/songs/domain/usecases/get_songs.dart';
import 'package:music_app/features/songs/presentation/bloc/song_bloc.dart';

final locator = GetIt.instance;

Future<void> init() async {
  //! BLoC
  locator.registerFactory(
    () => SongBloc(
      getLyrics: locator(),
      getSongs: locator(),
      getSong: locator(),
      songStorageRepository: locator(),
    ),
  );

  //! UseCases
  locator.registerLazySingleton(
    () => GetLyricsUseCase(songRepository: locator()),
  );
  locator.registerLazySingleton(
    () => GetSongsUseCase(songRepository: locator()),
  );
  locator.registerLazySingleton(
    () => GetSongUseCase(songRepository: locator()),
  );

  //! Repositories
  locator.registerLazySingleton<SongRepository>(
    () => SongRepositoryImplementation(
      networkInfo: locator(),
      remoteDataSource: locator(),
    ),
  );

  //! Data Source
  locator.registerLazySingleton<SongRemoteDataSource>(
    () => SongRemoteDataSourceImplementation(),
  );

  //! Core
  locator.registerLazySingleton<NetworkInfo>(
    () => NetworkInfoImpl(connectivity: locator()),
  );

  //! External Libraries
  locator.registerLazySingleton(() => Connectivity());

  //! Storage
  locator.registerLazySingleton(() => SongStorageRepository());
}
```

I have explained the use of locator, advantages and its methods in my previous blog on [dependency injection](https://sungod.hashnode.dev/dependency-injection) in flutter. 

The main thing to understand here is the use of locator() method. Let’s see, SongBloc needs three dependencies: lyrics useCase, song useCase and songsUsecase. But if you see the implementation of the useCases they also have some dependencies like song repository. And this chain goes on until we reach the data sources. So to avoid the duplication of code and object creation we use the locator() method.

The locator() method automatically searches the required dependency in the locator.dart file and replaces it in compile time. Meaning if we write locator() in front of getLyrics then we need to define the GetLyricsUseCase in the locator.dart file itself. So it becomes mandatory to list all dependencies in the locator file. In specially clean architecture their formation of chain in dependencies , if you observe -

BloC -&gt; UseCase -&gt; SongRepository -&gt; NetworkInfo and DataSource

After listing down all the dependencies we are done with dependency handling of our application. 

Now final part how to use this in UI, so go to main.dart file

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:music_app/core/repositories/song_storage.dart';
import 'package:music_app/features/songs/presentation/bloc/song_bloc.dart';
import 'package:music_app/features/songs/presentation/pages/songs_screen.dart';
import 'package:music_app/locator.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await init();
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MultiRepositoryProvider(
      providers: [
        RepositoryProvider(
          create: ((context) => locator<SongStorageRepository>()),
        ),
      ],
      child: MultiBlocProvider(
        providers: [
          BlocProvider(
            create: ((context) => locator<SongBloc>()),
          ),
        ],
        child: MaterialApp(
          debugShowCheckedModeBanner: false,
          home: Scaffold(body: SongsScreen()),
        ),
      ),
    );
  }
}
```

We use the locator object created in the locator.dart file to search any dependency of our application.

**And we are done ⭐ ⭐ ⭐.**

In conclusion, Clean Architecture serves as a practical and hands-on approach to developing Flutter applications. By incorporating distinct layers—Data, Domain, and Presentation—it enables a clear separation of concerns, enhancing maintainability, scalability, and testability.

**Congratulations on completing the Blog series !!!** 

**Keep Fluttering 🐦 🐦**