# Flutter BLoC Beginner's Guide Part - 2

Welcome to the second part of the Flutter BLoC for beginners series. In the last blog, we have seen the basics of BLoC like creation and usage, in this blog we are going to learn how to use Repositories in the Flutter BLoC project.

This is a continuation of part 1 so it’s recommended to read part 1 first,  so now let's get started.

Before going to actual implementation we need to understand why repositories ??

1. **Code separation**: Currently in our flutter project we are using only two layers i.e. BLoC layer and the Service layer. The service layer is responsible for API or Firebase calls and the rest of the data manipulation is done in the BLoC layer. Which is not feasible for larger projects and large no of events and states in one BLoC. So we are going to use 3 layers BLoC -&gt; Repository -&gt; Service.
    
2. **Data storage**: Repositories are also used for data storage, suppose on the home screen of our application we fetched all the users of the application and on the friend profile page we need to show the profile of one user. Well, we can counter this situation by passing the entire users list via routes but it's not a feasible solution when it's used in various locations. So one solution is using repositories, as I told we are going to use 3 layers so repository going to store this data and allow us to access this from anywhere in the application.
    

**Introduction** : 

So to understand Repositories we are creating a simple music app with Flutter BLoC.

Before that, we need free music API which will provide us the List of songs, for this

[https://developer.musixmatch.com/admin/applications](https://developer.musixmatch.com/admin/applications) register to this to get an API key which will be required for further application.

Now we have all the things needed to get started, first we are going to make SongBloc - 

```dart
part of 'song_bloc.dart';

@immutable
abstract class SongState extends Equatable {
  @override
  List<Object?> get props => [];
}

class SongLoadedState extends SongState {}

class SongLoadingState extends SongState {}

class SongLoadingFailure extends SongState {}
```

```dart
part of 'song_bloc.dart';

@immutable
abstract class SongEvent extends Equatable {
  @override
  List<Object?> get props => [];
}

class GetSongs extends SongEvent {}
```

```dart
import 'dart:developer';

import 'package:bloc/bloc.dart';
import 'package:equatable/equatable.dart';
import 'package:meta/meta.dart';

import '../../Repository/song_repository.dart';

part 'song_event.dart';
part 'song_state.dart';

class SongBloc extends Bloc<SongEvent, SongState> {
  final SongRepository _repository;
  SongBloc({required SongRepository songRepository})
      : _repository = songRepository,
        super(SongLoadingState()) {
    on<GetSongs>(_getSongs);
  }

  Future<void> _getSongs(GetSongs event, Emitter<SongState> emit) async {
    emit(SongLoadingState());
    try {
      await _repository.loadSongs();
      emit(SongLoadedState());
    } catch (e) {
      log(e.toString());
      emit(SongLoadingFailure());
    }
  }
}
```

I have already explained the basics of the bloc in my previous bloc, just change we are using loadSongs() function from the repository, we will see that in a minute. Make a note here we are not returning data in any states like we normally do in Bloc.

Now we write our repository class -

```dart
import 'dart:convert';

import 'package:dio/dio.dart';
import '../Services/song_service.dart';

import '../Models/song_model.dart';

class SongRepository {
  final SongService _service = SongService();
  List<Song> songs = [];

  Future<void> loadSongs() async {
    Response? response = await _service.getSongs();
    if (response == null) return;
    songs.clear();
    Map<String, dynamic> res = jsonDecode(response.data);
    for (var element in res["message"]["body"]["track_list"]) {
      songs.add(Song.fromJson(element["track"]));
    }
  }

  String getSongNameByID(int id) {
    for (Song song in songs) {
      if (song.songId == id) return song.songName;
    }
    return "";
  }

  String getArtistNameByID(int id) {
    for (Song song in songs) {
      if (song.songId == id) return song.artistName;
    }
    return "";
  }
}
```

In this, we have declared a list of songs that will store songs and we have made three functions. The first is to fetch songs from API and the others are to just retrieve song name and artist name by using songID, which will be used in the second screen.

We have used a model class Song - 

```dart
class Song {
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

  factory Song.fromJson(Map<String, dynamic> json) {
    return Song(
      songId: json["track_id"],
      songName: json["track_name"],
      albumId: json["album_id"],
      albumName: json["album_name"],
      artistName: json["artist_name"],
    );
  }
}
```

In the repository, we have used getSongs() functions which are defined in SongServiceClass -

```dart
import 'package:dio/dio.dart';
import 'package:flutter/cupertino.dart';

class SongService {
  late Dio dio;

  SongService() {
    dio = Dio();
  }

  Future getSongs() async {
    try {
      String url =
          "https://api.musixmatch.com/ws/1.1/chart.tracks.get?apikey=8dbbbf65ba63d8e5278851222fc09948";
      final response = await dio.get(url);
      return response;
    } on DioError catch (e) {
      debugPrint("Status Code : ${e.response!.statusCode.toString()}");
    }
  }
}
```

Now we have completed both the Bloc and Repository parts ready.

Let’s register this in the main.dart -

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:music_app/Repository/song_repository.dart';

import 'BLoC/song bloc/song_bloc.dart';
import 'Presentation/home.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MultiRepositoryProvider(
      providers: [
        RepositoryProvider(create: (context) => SongRepository()),
      ],
      child: MultiBlocProvider(
        providers: [
          BlocProvider(
            create: ((context) =>
                SongBloc(songRepository: context.read<SongRepository>())
                  ..add(GetSongs())),
          ),
        ],
        child: MaterialApp(
          debugShowCheckedModeBanner: false,
          home: Scaffold(body: HomeScreen()),
        ),
      ),
    );
  }
}
```

The new thing in main.dart is MultiRepositoryProvider - 

We need to register our repository at the top level to be accessible from anywhere in applications. 

It is very similar to MultiBlocProvider but it's for repositories.

Now let's make our UI - 

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:music_app/Models/song_model.dart';
import 'package:music_app/Presentation/errorpage.dart';
import 'package:music_app/Presentation/song_details_page.dart';
import 'package:music_app/Repository/song_repository.dart';

import '../BLoC/song bloc/song_bloc.dart';

class SongsScreen extends StatelessWidget {
  const SongsScreen({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: BlocBuilder<SongBloc, SongState>(
        builder: (context, state) {
          if (state is SongLoadingState)
            return Center(child: CircularProgressIndicator());
          if (state is SongLoadedState) {
            return ListView.builder(
              itemCount: context.read<SongRepository>().songs.length,
              itemBuilder: (context, index) {
                Song song = context.read<SongRepository>().songs[index];
                return ListTile(
                  style: ListTileStyle.drawer,
                  isThreeLine: false,
                  title: Text(song.songName),
                  subtitle: Text("From ${song.albumName}"),
                  leading: Image.asset("assets/images/music.png"),
                  onTap: () {
                    Navigator.push(
                      context,
                      MaterialPageRoute(
                          builder: (context) => SongDetails(id: song.songId)),
                    );
                  },
                );
              },
            );
          } else
            return ErrorPage(errorMessage: "Something Went Wrong");
        },
      ),
    );
  }
}
```

To access the data from the repository we need to use the following syntax - 

[context.read](http://context.read)&lt;REPOSITORY\_NAME&gt;();

What this does is it returns the already created instance of SongRepository i.e. we have done in main.dart.

So we are done with the first page, now the second screen is going to be a song details screen where we will show appropriate details according to song ID.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:music_app/Repository/song_repository.dart';

class SongDetails extends StatelessWidget {
  final int id;
  SongDetails({Key? key, required this.id}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Music App"),
        backgroundColor: Colors.teal.withOpacity(0.5),
      ),
      body: SingleChildScrollView(
        child: Padding(
          padding: EdgeInsets.all(15),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.center,
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Image.asset(
                "assets/images/play.jpg",
              ),
              Text(
                context.read<SongRepository>().getSongNameByID(id),
                style: TextStyle(fontSize: 24),
              ),
              Text(
                "by ${context.read<SongRepository>().getArtistNameByID(id)}",
                style: TextStyle(fontSize: 18),
              ),
              SizedBox(height: 20),
            ],
          ),
        ),
      ),
    );
  }
}
```

And that’s it !!!

The source code for this article is here:-

[https://github.com/PranavMasekar/Music-App/tree/blog-example](https://github.com/PranavMasekar/Music-App/tree/blog-example)

**Congratulations !!** You have implemented repositories in your flutter BLoC project.