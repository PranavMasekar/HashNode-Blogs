---
title: "Integrating HealthKit with Flutter"
datePublished: Wed Oct 15 2025 04:30:27 GMT+0000 (Coordinated Universal Time)
cuid: cmgrht5hk001402ju28gj4ik6
slug: healthkit
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1760004564324/4400eea4-a6e6-4101-88a5-79da37117f35.png
tags: swift, flutter, healthtech, swiftui

---

## Introduction

If you're a Flutter developer building health and fitness apps, you know the struggle! Getting access to native iOS health data like steps, calories, and heart rate can feel like navigating a maze of native code integration. But trust me, with Pigeon and a bit of Swift magic, we can make this happen seamlessly.

## Prerequisites

Before we dive in, make sure you have these things ready:

* Flutter SDK (3.9.0+)
    
* Xcode with iOS Simulator or physical iPhone
    
* Basic understanding of Flutter and iOS development
    

## What is HealthKit?

**HealthKit** is Apple's framework for health and fitness data. It provides a centralised repository for health information from various sources like the iPhone, Apple Watch, and third-party apps. Think of it as the single source of truth for all your health metrics.

**Pigeon** is Flutter's code generation tool that creates type-safe communication between Flutter and native platforms. If you're new to Pigeon, check out my [complete Pigeon guide](https://sungod.hashnode.dev/pigeon) for a deep dive into how it works!

## Let's start with the Pigeon API definition

First things first, we need to define our API contract. Create a new file in the `pigeons` directory:

```Dart
import 'package:pigeon/pigeon.dart';

@ConfigurePigeon(
  PigeonOptions(
    dartOut: 'lib/healthkit_api.dart',
    dartOptions: DartOptions(),
    kotlinOut:
        'android/app/src/main/kotlin/com/pranav/iosPlayground/HealthKitApi.kt',
    kotlinOptions: KotlinOptions(),
    swiftOut: 'ios/Runner/HealthKitApi.swift',
    swiftOptions: SwiftOptions(),
    dartPackageName: 'com.pranav.iosPlayground',
  ),
)
class HealthSummary {
  int steps;
  double calories;
  double avgHeartRate;

  HealthSummary({
    required this.steps,
    required this.calories,
    required this.avgHeartRate,
  });
}

@HostApi()
abstract class HealthKitApi {
  @async
  HealthSummary getHealthSummary();
}
```

What's happening here:

* We define a `HealthSummary` class that holds our health metrics
    
* The `@HostApi()` annotation tells Pigeon this is our main API interface
    
* `@async` makes the method asynchronous (obviously!)
    
* Pigeon will generate all the boilerplate code for us
    

## Add Pigeon to your pubspec.yaml

Add Pigeon as a dev dependency:

```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^5.0.0
  pigeon: ^26.0.1
```

## Generate the platform code

Now here's the exciting part! Run this command to generate all the platform-specific code:

```bash
dart run pigeon --input pigeons/healthkit_api.dart
```

This single command generates:

* Dart API client (`lib/healthkit_api.dart`)
    
* Swift protocol and setup code (`ios/Runner/HealthKitApi.swift`)
    
* Kotlin interface (for Android, though we're focusing on iOS here)
    

## Implement the Swift HealthKit logic

Create the Swift implementation that actually talks to HealthKit:

```swift
import Foundation
import HealthKit

class HealthKitApiImplementation: HealthKitApi {

    let store = HKHealthStore()
    
    let typesToRead: Set = [
        HKObjectType.quantityType(forIdentifier: .stepCount)!,
        HKObjectType.quantityType(forIdentifier: .activeEnergyBurned)!,
        HKObjectType.quantityType(forIdentifier: .heartRate)!
    ]
    
    func requestPermission(completion: @escaping (Bool) -> Void) {
        store.requestAuthorization(toShare: [], read: typesToRead) { success, error in
            if success {
                print("Permission Approved")
                completion(true)
            } else {
                print("Permission Rejected: \(String(describing: error))")
                completion(false)
            }
        }
    }
    
    func getHealthSummary(completion: @escaping (Result<HealthSummary, any Error>) -> Void) {
        requestPermission { granted in
            guard granted else {
                completion(.success(HealthSummary(steps: 0, calories: 0, avgHeartRate: 0)))
                return
            }
            
            let startOfDay = Calendar.current.startOfDay(for: Date())
            let predicate = HKQuery.predicateForSamples(withStart: startOfDay, end: Date(), options: .strictStartDate)
            
            var steps: Double = 0
            var calories: Double = 0
            var heartRate: Double = 0
            
            let group = DispatchGroup()
            
            if let type = HKQuantityType.quantityType(forIdentifier: .stepCount) {
                group.enter()
                let query = HKStatisticsQuery(quantityType: type, quantitySamplePredicate: predicate, options: .cumulativeSum) { _, result, _ in
                    steps = result?.sumQuantity()?.doubleValue(for: .count()) ?? 0
                    group.leave()
                }
                self.store.execute(query)
            }
            
            if let type = HKQuantityType.quantityType(forIdentifier: .activeEnergyBurned) {
                group.enter()
                let query = HKStatisticsQuery(quantityType: type, quantitySamplePredicate: predicate, options: .cumulativeSum) { _, result, _ in
                    calories = result?.sumQuantity()?.doubleValue(for: .kilocalorie()) ?? 0
                    group.leave()
                }
                self.store.execute(query)
            }
            
            if let type = HKQuantityType.quantityType(forIdentifier: .heartRate) {
                group.enter()
                let query = HKStatisticsQuery(quantityType: type, quantitySamplePredicate: predicate, options: .discreteAverage) { _, result, _ in
                    heartRate = result?.averageQuantity()?.doubleValue(for: HKUnit(from: "count/min")) ?? 0
                    group.leave()
                }
                self.store.execute(query)
            }
            
            group.notify(queue: .main) {
                let summary = HealthSummary(
                    steps: Int64(steps),
                    calories: calories,
                    avgHeartRate: heartRate
                )
                print("Health summary:", summary)
                completion(.success(summary))
            }
        }
    }
}
```

Key points here:

* We request permissions for steps, calories, and heart rate data
    
* `DispatchGroup` helps us wait for all three queries to complete
    
* We query today's data using `predicateForSamples`
    
* Each query uses different options: `.cumulativeSum` for totals, `.discreteAverage` for averages
    

## Wire it into the iOS app

Update your `AppDelegate.swift` to register our implementation:

```Swift
    let healthKitApiImplementation = HealthKitApiImplementation()
        
    let controller = window?.rootViewController as! FlutterViewController
        
    HealthKitApiSetup.setUp(
        binaryMessenger: controller.binaryMessenger,
        api: healthKitApiImplementation
    )
```

## Add HealthKit permissions to Info.plist

Don't forget the privacy descriptions! Add these to your `Info.plist`:

```Plist
	<key>NSHealthShareUsageDescription</key>
	<string>This app reads your Health data (steps, calories, heart rate) to show your activity summary.</string>
	<key>NSHealthUpdateUsageDescription</key>
	<string>This app updates your Health data when needed to keep your activity information accurate.</string>
```

## Create the Flutter UI

Now let's build a beautiful UI to display our health data:

```dart
import 'package:flutter/material.dart';
import 'package:shadcn_ui/shadcn_ui.dart';

import 'healthkit_api.dart';

class HomeView extends StatelessWidget {
  const HomeView({super.key});

  Future<HealthSummary> _loadSummary() {
    final api = HealthKitApi();
    return api.getHealthSummary();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        surfaceTintColor: Colors.transparent,
        title: Text(
          'Health Summary',
          style: ShadTheme.of(context).textTheme.h2,
        ),
        centerTitle: true,
      ),
      body: SafeArea(
        child: Padding(
          padding: const EdgeInsets.all(16),
          child: FutureBuilder<HealthSummary>(
            future: _loadSummary(),
            builder: (context, snapshot) {
              if (snapshot.connectionState == ConnectionState.waiting) {
                return const Center(child: CircularProgressIndicator());
              }
              if (snapshot.hasError) {
                return Center(
                  child: Text(
                    'Failed to load health data',
                    style: ShadTheme.of(context).textTheme.muted,
                  ),
                );
              }
              final data = snapshot.data;
              if (data == null) {
                return Center(
                  child: Text(
                    'No data available',
                    style: ShadTheme.of(context).textTheme.muted,
                  ),
                );
              }

              return Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Expanded(
                    child: GridView.count(
                      crossAxisCount: 2,
                      mainAxisSpacing: 16,
                      crossAxisSpacing: 16,
                      childAspectRatio: 1.25,
                      children: [
                        _MetricCard(
                          title: 'Steps',
                          value: data.steps.toString(),
                          subtitle: 'Today',
                        ),
                        _MetricCard(
                          title: 'Calories',
                          value: data.calories.toStringAsFixed(0),
                          subtitle: 'kcal',
                        ),
                        _MetricCard(
                          title: 'Avg Heart Rate',
                          value: data.avgHeartRate.toStringAsFixed(0),
                          subtitle: 'bpm',
                        ),
                      ],
                    ),
                  ),
                ],
              );
            },
          ),
        ),
      ),
    );
  }
}
```

What's happening here:

* We use `FutureBuilder` to handle the async health data loading
    
* The UI shows loading, error, and success states (pro tip: always handle these!)
    
* Grid layout displays our health metrics in beautiful cards
    
* We format numbers appropriately (no decimals for steps, one decimal for heart rate)
    

## Test it out!

Run your app on an iOS device or simulator:

```bash
flutter run
```

When you first launch the app, iOS will prompt you to grant HealthKit permissions. Once approved, you'll see your health data displayed in a clean, modern UI!

## Wrap-up

So there you have it! We created a complete HealthKit integration using Pigeon for type-safe communication, implemented native Swift code to access health data, and built a beautiful Flutter UI to display it all.

You can find the complete source code for this project in the [GitHub repository](https://github.com/PranavMasekar/ios_playground/tree/healthkit) - feel free to fork it and experiment with your own health metrics!

**Keep Fluttering 💙💙💙**