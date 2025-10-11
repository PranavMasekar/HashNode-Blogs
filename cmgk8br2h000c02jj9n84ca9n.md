---
title: "Swift Integration via Pigeon"
seoTitle: "Pigeon: Streamlining Swift Integration"
seoDescription: "Learn how to use Pigeon to integrate Swift haptics in Flutter, ensuring type-safe API calls and streamlined native setup"
datePublished: Fri Oct 10 2025 02:30:36 GMT+0000 (Coordinated Universal Time)
cuid: cmgk8br2h000c02jj9n84ca9n
slug: pigeon
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1759813249347/71dbf155-b6a3-4c60-8aea-81167a96191e.png
tags: dart, swift, ios, flutter

---

## Introduction

Platform channels let Flutter call native APIs when you need capabilities outside the framework. **Pigeon** is a tool that generates the type-safe plumbing for those calls, so you focus on APIs instead of bytes and codecs. In this post, we’ll create a tiny Pigeon API for iOS haptics, implement it in Swift, and trigger it from Flutter buttons.

## Prerequisites

* Flutter SDK installed
    
* Xcode set up for iOS builds
    

## What is Pigeon?

Pigeon generates matching Dart and host-platform (Swift/Kotlin) code from a single schema. You declare an API interface once; Pigeon outputs:

* Dart client that sends typed messages
    
* Swift/Kotlin protocol + setup glue to receive those messages
    

This eliminates manual `MethodChannel` boilerplate and reduces runtime errors.

## Define the API (Pigeon schema)

Create `pigeons/haptics.dart` describing a host API that iOS will implement. Ours exposes one method: `triggerHapticFeedback(String type)`.

```dart
import 'package:pigeon/pigeon.dart';

@ConfigurePigeon(
  PigeonOptions(
    dartOut: 'lib/haptics.dart',
    dartOptions: DartOptions(),
    kotlinOut:
        'android/app/src/main/kotlin/com/pranav/iosPlayground/Haptics.kt',
    kotlinOptions: KotlinOptions(),
    swiftOut: 'ios/Runner/Haptics.swift',
    swiftOptions: SwiftOptions(),
    dartPackageName: 'com.pranav.iosPlayground',
  ),
)
@HostApi()
abstract class HapticsApi {
  void triggerHapticFeedback(String type);
}
```

The `@ConfigurePigeon` annotation tells Pigeon where to place generated files for Dart, Swift, and (optionally) Android. We’ll focus on Flutter and Swift here.

### **Why an abstract** `HapticsApi`?

* Pigeon uses an abstract class as the schema for your API. Methods are declarations (no bodies) that describe the shape of calls Flutter will make to the host platform.
    
* `@HostApi` marks this interface as implemented on the host (iOS in our case). Pigeon will generate:
    
    * A concrete Dart client named `HapticsApi` that sends typed messages.
        
    * A Swift protocol `HapticsApi` plus a setup helper to receive those messages.
        
* You do not implement this Dart abstract class yourself. Instead, you implement the generated Swift protocol (`HapticsApi`) on iOS and register it.
    

## Add Pigeon to dev\_dependencies

```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^5.0.0
  pigeon: ^26.0.1
```

Then generate code:

```bash
dart run pigeon --input pigeons/haptics.dart
```

This creates a Dart client (`lib/haptics.dart`) and Swift protocol/setup (`ios/Runner/Haptics.swift`).

## Implement the API in Swift

Pigeon generates a Swift `HapticsApi` protocol. We implement it and choose a `UIImpactFeedbackGenerator` style based on the requested type.

```swift
class HapticsImplementation: HapticsApi {
    func triggerHapticFeedback(type: String) {
        let impactStyle: UIImpactFeedbackGenerator.FeedbackStyle
        
        switch type {
        case "Light":
            impactStyle = .light
        case "Medium":
            impactStyle = .medium
        case "Heavy":
            impactStyle = .heavy
        default:
            impactStyle = .medium
        }
        
        let generator = UIImpactFeedbackGenerator(style: impactStyle)
        generator.impactOccurred()
    }
}
```

Register the implementation with the generated setup so it can receive messages from Flutter.

```swift
    GeneratedPluginRegistrant.register(with: self)
      
    let hapticsAPI = HapticsImplementation()
      
    let controller = window?.rootViewController as! FlutterViewController
      
    HapticsApiSetup.setUp(binaryMessenger: controller.binaryMessenger, api: hapticsAPI)
      
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
```

That’s all the native setup. No manual channel names or codecs needed Pigeon generated them.

## Call it from Flutter

Use the generated Dart client to invoke haptics from UI. Here, three buttons trigger light/medium/heavy feedback.

```dart
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                ShadButton(
                  onPressed: () async {
                    await HapticsApi().triggerHapticFeedback('Light');
                  },
                  child: const Text('Light'),
                ),
                const SizedBox(width: 12),
                ShadButton(
                  onPressed: () async {
                    await HapticsApi().triggerHapticFeedback('Medium');
                  },
                  child: const Text('Medium'),
                ),
                const SizedBox(width: 12),
                ShadButton(
                  onPressed: () async {
                    await HapticsApi().triggerHapticFeedback('Heavy');
                  },
                  child: const Text('Heavy'),
                ),
              ],
            ),
```

## Run it

Build and run on an iOS device or simulator. Tap the buttons to feel the feedback.

```bash
flutter run -d ios
```

## Troubleshooting: `Haptics.swift` not visible in Xcode

If the generated `Haptics.swift` doesn’t appear in Xcode’s Project Navigator even though it exists on disk, you can work around it by creating the file in Xcode and pasting the generated contents:

* In Xcode, open the `Runner` project.
    
* Right‑click the `Runner` group → New File… → Swift File.
    
* Name it exactly `Haptics.swift` and save it under `ios/Runner/`.
    
* Open the generated file on disk (`ios/Runner/Haptics.swift`) and copy all contents into the newly created file in Xcode.
    
* In the File Inspector, ensure Target Membership includes `Runner`.
    
* Clean and rebuild:
    

```dart
flutter clean && flutter pub get && flutter run -d ios
```

If you regenerate with Pigeon and Xcode loses the reference again, repeat these steps.

## Wrap-up

We defined a tiny Pigeon API, generated Dart/Swift glue, implemented a Swift handler for `UIImpactFeedbackGenerator`, and invoked it from Flutter. Pigeon keeps platform channels type-safe and ergonomic, so you can focus on features instead of plumbing.

Github Repo: \[[Link](https://github.com/PranavMasekar/ios_playground/tree/haptics)\]

**Keep Fluttering 💙💙💙**