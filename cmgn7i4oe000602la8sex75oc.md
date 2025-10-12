---
title: "Add AirPlay to Flutter in 5 Minutes"
seoTitle: "Quick AirPlay Setup for Flutter Apps"
seoDescription: "Add AirPlay to your Flutter app in 5 minutes. Implement seamless streaming with Pigeon and native Swift code for a beautiful audio player"
datePublished: Sun Oct 12 2025 04:30:52 GMT+0000 (Coordinated Universal Time)
cuid: cmgn7i4oe000602la8sex75oc
slug: airplay
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1760004134853/20e45e76-d97c-4144-b86b-cabb42f43702.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1760244430542/31a17573-c9e9-4cca-bdbd-6e57771b618b.png
tags: dart, swift, flutter, swiftui

---

## Introduction

AirPlay is Apple's wireless streaming protocol that lets users stream audio and video from their iOS devices to Apple TV, HomePod, and other AirPlay-compatible devices. In this post, we'll implement AirPlay functionality in a Flutter app using Pigeon for seamless communication between Flutter and native Swift code, then wire it into a beautiful audio player interface.

## Prerequisites

* Flutter project with iOS support
    
* Xcode for iOS development
    
* Basic understanding of Swift and Flutter
    
* Audio streaming requirements (just\_audio package)
    

## Prerequisites for Pigeon

If you're not familiar with Pigeon, check out our [comprehensive Pigeon guide](https://sungod.hashnode.dev/pigeon) that covers setup, configuration, and best practices for Flutter-to-native communication.

### Let's start with the Pigeon API definition

First, create the Pigeon API definition that will generate our communication layer:

```dart
import 'package:pigeon/pigeon.dart';

@ConfigurePigeon(
  PigeonOptions(
    dartOut: 'lib/airplay_api.dart',
    dartOptions: DartOptions(),
    kotlinOut:
        'android/app/src/main/kotlin/com/example/ios_playground/Airplay.kt',
    kotlinOptions: KotlinOptions(),
    swiftOut: 'ios/Runner/Airplay.swift',
    swiftOptions: SwiftOptions(),
    dartPackageName: 'com.example.ios_playground',
  ),
)
@HostApi()
abstract class AirplayApi {
  void startAirplay();
  void stopAirplay();
}
```

**Key points:**

* `@HostApi()` marks this as a Flutter-to-native API
    
* `dartOut`, `swiftOut`, `kotlinOut` specify where generated files go
    
* Simple void methods for starting and stopping AirPlay
    

### Add Pigeon to your project

Add Pigeon as a dev dependency in your `pubspec.yaml`:

```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^5.0.0
  pigeon: ^26.0.1
```

### Generate the Pigeon code

Run the Pigeon code generation command:

```bash
dart run pigeon --input pigeons/airplay_api.dart
```

This creates three files:

* `lib/airplay_api.dart` - Flutter-side API
    
* `ios/Runner/Airplay.swift` - Swift protocol and setup
    
* Android Kotlin files (for completeness)
    

## Implement the Swift AirPlay functionality

Create `AirplayImplementation.swift` with native AirPlay controls:

```swift
import Foundation
import AVKit
import MediaPlayer
import UIKit

class AirplayImplementation: AirplayApi {
    
    private var routePickerView: AVRoutePickerView?
    
    func startAirplay() throws {
        DispatchQueue.main.async {
            // Setup audio session for AirPlay playback
            let session = AVAudioSession.sharedInstance()
            do {
                print("Setting audio session.....")
                try session.setCategory(.playback, mode: .default, options: [.allowAirPlay])
                try session.setActive(true)
            } catch {
                print("Failed to configure AVAudioSession: \(error)")
            }
            
            // Add AirPlay picker programmatically (hidden button)
            print("Showing Airplay Picker.....")
            let routePicker = AVRoutePickerView(frame: .zero)
            routePicker.isHidden = true // no visible UI
            UIApplication.shared.windows.first?.rootViewController?.view.addSubview(routePicker)
            
            self.routePickerView = routePicker
            
            // Programmatically trigger the AirPlay device selector
            if let button = routePicker.subviews.compactMap({ $0 as? UIButton }).first {
                button.sendActions(for: .touchUpInside)
            }
        }
    }
    
    func stopAirplay() throws {
        DispatchQueue.main.async {
            let session = AVAudioSession.sharedInstance()
            do {
                // Reset session so playback goes back to device speakers
                try session.setActive(false, options: [.notifyOthersOnDeactivation])
            } catch {
                print("Failed to stop AirPlay: \(error)")
            }
        }
    }
}
```

**What this does:**

* `AVAudioSession` configuration enables AirPlay routing
    
* `AVRoutePickerView` provides the native AirPlay device picker
    
* Programmatic button tap triggers the AirPlay selection UI
    
* `stopAirplay()` resets audio routing back to device speakers
    

### Register the implementation in AppDelegate

Wire the Swift implementation into your Flutter app:

```swift
import Flutter
import UIKit

@main
@objc class AppDelegate: FlutterAppDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    GeneratedPluginRegistrant.register(with: self)
      
    let airplayImplementation = AirplayImplementation()
                
    let controller = window?.rootViewController as! FlutterViewController
                
    AirplayApiSetup.setUp(binaryMessenger: controller.binaryMessenger, api: airplayImplementation)
      
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }
}
```

**Key steps:**

* Create instance of `AirplayImplementation`
    
* Get the Flutter view controller
    
* Use `AirplayApiSetup.setUp()` to register the implementation
    

## Create the Flutter UI

Build a beautiful audio player with AirPlay integration. Here are the key UI components:

**AirPlay button integration:**

```dart
ShadButton.secondary(
  onPressed: () {
    AirplayApi().startAirplay();
  },
  child: const Icon(Icons.airplay_rounded, size: 28),
),
```

**Audio player setup:**

```dart
// Initialize audio player
late AudioPlayer _audioPlayer;
bool _isPlaying = false;

// Load audio source
await _audioPlayer.setAudioSource(
  AudioSource.uri(Uri.parse('your-audio-url')),
);
```

### Test the AirPlay functionality

1. **Run on real iOS device**
    
2. **Tap the AirPlay button** - native device picker appears
    
3. **Select an AirPlay device** - audio routes automatically
    
4. **Play audio** - streams to selected device
    

## Wrap-up

We successfully implemented AirPlay functionality in Flutter using Pigeon for type-safe communication and native Swift code for AirPlay controls. The solution provides seamless audio streaming to AirPlay devices with a clean, native user experience.

The complete implementation is available in the [iOS Playground repository](https://github.com/PranavMasekar/ios_playground/tree/airplay) with working examples.

**Keep Fluttering 💙💙💙**