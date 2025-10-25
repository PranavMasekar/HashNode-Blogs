---
title: "How to Use Apple's Vision (Swift) with Flutter for Text Detection in Images"
datePublished: Sat Oct 25 2025 04:30:39 GMT+0000 (Coordinated Universal Time)
cuid: cmh5s7wrk000402le1qr0auva
slug: apples-vision-swift-with-flutter
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1760968795884/336d8fd4-a6b6-47f3-82db-2cb53c596366.png
tags: dart, swift, flutter, ocr, swiftui

---

## Introduction

If you're a Flutter dev building for iOS, you know the struggle! OCR is super useful (receipts, IDs, notes, you name it), but wiring native Vision in Swift to Flutter can feel like glue-code (pain!). In this post, we'll integrate Apple's Vision framework into a Flutter app using Pigeon for type-safe platform channels and build a tiny UI to pick an image and extract text—end to end.

* **Repo**: [`iOS Playground`](https://github.com/PranavMasekar/ios_playground)
    
* **What we'll build**: Pick an image in Flutter → call native Swift → run `VNRecognizeTextRequest` → display extracted text.
    
* **Why Pigeon?** Type-safe messages, autogenerates Swift/Dart stubs, and removes dynamic channel string juggling (obviously!).
    

## Prerequisites

* Xcode + iOS Simulator (or a real device)
    
* Flutter SDK (stable)
    
* Basic Swift familiarity (just enough to read the Vision request)
    
* Packages used in UI: `image_picker`, `shadcn_ui`
    

## Define the API with Pigeon

We start with a tiny, host-side API in Pigeon. This generates Dart, Swift, and Kotlin stubs for us.

```dart
import 'package:pigeon/pigeon.dart';

@ConfigurePigeon(
  PigeonOptions(
    dartOut: 'lib/vision_api.dart',
    dartOptions: DartOptions(),
    kotlinOut:
        'android/app/src/main/kotlin/com/example/ios_playground/VisionApi.kt',
    kotlinOptions: KotlinOptions(),
    swiftOut: 'ios/Runner/VisionApi.swift',
    swiftOptions: SwiftOptions(),
    dartPackageName: 'com.pranav.iosPlayground',
  ),
)
@HostApi()
abstract class VisionApi {
  @async
  String? detectText(String image);
}
```

* `@HostApi`: Dart will call into the host (iOS) side.
    
* `detectText` takes an image file path and returns the recognized string (nullable).
    
* Outputs are configured to write the generated code into `lib/` and `ios/Runner/`.
    

### Generate code with Pigeon

Run this to generate the Dart/Swift/Kotlin stubs based on the configuration above:

```bash
dart run pigeon --input pigeons/vision_api.dart
```

This command reads the `@ConfigurePigeon` annotation and writes the outputs to the paths you saw in the snippet.

## Implement Vision in Swift

Now the fun part—implementing text recognition using Vision. Here's the concrete implementation that does the OCR on iOS:

```swift
class VisionApiImplementation: VisionApi {
    func detectText(image: String, completion: @escaping (Result<String?, any Error>) -> Void) {
        // 1. Load image from file path
        guard let uiImage = UIImage(contentsOfFile: image),
              let cgImage = uiImage.cgImage else {
            completion(.failure(NSError(domain: "Invalid image path", code: -1)))
            return
        }

        // 2. Create Vision text recognition request
        let request = VNRecognizeTextRequest { request, error in
            if let error = error {
                completion(.failure(error))
                return
            }

            // 3. Extract recognized text
            guard let observations = request.results as? [VNRecognizedTextObservation] else {
                completion(.success(nil))
                return
            }

            let recognizedStrings = observations.compactMap { observation in
                observation.topCandidates(1).first?.string
            }

            // 4. Join text lines into single string
            let fullText = recognizedStrings.joined(separator: "\n")
            completion(.success(fullText))
        }

        // Configure request for accuracy
        request.recognitionLevel = .accurate
        request.usesLanguageCorrection = true

        // 5. Perform the request asynchronously
        let handler = VNImageRequestHandler(cgImage: cgImage, options: [:])
        DispatchQueue.global(qos: .userInitiated).async {
            do {
                try handler.perform([request])
            } catch {
                completion(.failure(error))
            }
        }
    }
}
```

What this does:

* It loads the image using `UIImage(contentsOfFile:)` and extracts a `CGImage`. If the path is invalid, it returns an error back to Flutter.
    
* It creates a `VNRecognizeTextRequest` and, in its completion handler, collects the top string candidate from each `VNRecognizedTextObservation`.
    
* It joins all recognized lines with newlines and returns a single string.
    
* It opts into higher quality with `recognitionLevel = .accurate` and enables language correction for better results.
    
* It performs the Vision request off the main thread to keep the UI responsive, then reports success or failure via the completion callback.
    

## Wire It Up in AppDelegate (iOS)

Connect the generated messaging to your implementation during app launch:

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
      
    let visionApiImplementation = VisionApiImplementation()
      
    let controller = window?.rootViewController as! FlutterViewController
      
    VisionApiSetup.setUp(binaryMessenger: controller.binaryMessenger, api: visionApiImplementation)

    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }
}
```

* Instantiates your `VisionApiImplementation`.
    
* Grabs the `FlutterViewController` and its `binaryMessenger`.
    
* Registers the handler via `VisionApiSetup.setUp(...)` so Dart calls reach Swift.
    

## Flutter UI: Pick an Image and Display Text

Finally, wire the call from the Flutter side. The sample UI uses `image_picker` to select an image and shows a progress indicator while the iOS side runs OCR.

```dart
class _HomeViewState extends State<HomeView> {
  final VisionApi _visionApi = VisionApi();
  final ImagePicker _picker = ImagePicker();
  File? _selectedImage;
  String? _extractedText;
  bool _isProcessing = false;

  Future<void> _pickImage() async {
    try {
      final XFile? image = await _picker.pickImage(
        source: ImageSource.gallery,
        maxWidth: 1024,
        maxHeight: 1024,
        imageQuality: 85,
      );

      if (image != null) {
        setState(() {
          _selectedImage = File(image.path);
          _extractedText = null;
        });

        await _extractText(image.path);
      }
    } catch (e) {
      if (mounted) {
        ScaffoldMessenger.of(
          context,
        ).showSnackBar(SnackBar(content: Text('Error picking image: $e')));
      }
    }
  }

  Future<void> _extractText(String imagePath) async {
    setState(() {
      _isProcessing = true;
    });

    try {
      final String? text = await _visionApi.detectText(imagePath);
      setState(() {
        _extractedText = text;
        _isProcessing = false;
      });
    } catch (e) {
      setState(() {
        _isProcessing = false;
      });
      if (mounted) {
        ScaffoldMessenger.of(
          context,
        ).showSnackBar(SnackBar(content: Text('Error extracting text: $e')));
      }
    }
  }
}
```

## Running It

1. Open the repo and get packages:
    

```bash
flutter pub get
```

2. Run on iOS Simulator (or device):
    

```bash
flutter run -d ios
```

3. Tap “Pick Image”, choose an image with text, and watch the text appear. 🎉
    

## Notes and Tips

* Vision works on-device and is fast for clean, high-contrast text. For tricky cases, try different lighting or preprocessing.
    
* You can switch `recognitionLevel` to `.fast` for speed, or `.accurate` (as used here) for quality.
    
* If you add languages, consider `recognitionLanguages` on `VNRecognizeTextRequest`.
    
* Error handling matters—this sample returns failures back to Flutter as errors; show a user-friendly message.
    

## Wrap-up

So there you have it! We defined a tiny Pigeon API, generated type-safe messaging, implemented Vision text recognition in Swift, and wired it to a clean Flutter UI. The result: a smooth, native OCR flow with minimal boilerplate.

* Full code in the repo: [`iOS Playground`](https://github.com/PranavMasekar/ios_playground)
    
* Try extending it: add camera capture, multi-language recognition, or bounding boxes overlay.
    

**Keep Fluttering 💙💙💙**