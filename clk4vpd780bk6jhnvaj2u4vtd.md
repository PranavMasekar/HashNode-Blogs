---
title: "Flutter Extensions: A Must-Have for Any Developer"
datePublished: Sun Jul 16 2023 03:30:08 GMT+0000 (Coordinated Universal Time)
cuid: clk4vpd780bk6jhnvaj2u4vtd
slug: flutter-extensions
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1689435426418/ba5b2a70-5d03-4bec-81d0-e67cf53e7444.png
tags: flutter, community, extensions, wemakedevs, dry-principles

---

## Introduction :

Flutter extensions are a powerful tool that can be used to extend the functionality of Flutter apps. They allow developers to add new features, improve performance, and simplify their code. There are a wide variety of Flutter extensions available, so developers can find one that fits their specific needs. In this blog post, we will discuss the basics of Flutter extensions. We will also share some of our favourite Flutter extensions that we use in our projects. By the end of this post, you will know how to use Flutter extensions to improve your Flutter apps.

## What exactly are Flutter Extensions?

Flutter extensions are a way to add new functionality to existing widgets in Flutter. They are similar to mixins in other programming languages, but they are more lightweight and easier to use. Flutter extensions are a powerful way to add new functionality to existing widgets. They are a great way to avoid duplicating code and to make your Flutter code more modular and reusable. This also makes code more user-friendly and more readable to other developers.

### String Extensions

```dart
String capitalizeFirstLetter() {
    if (isNullOrEmpty) {
      return '';
    }
    if (this!.length == 1) {
      return this![0].toUpperCase();
    }
    return "${this![0].toUpperCase()}${this!.substring(1).toLowerCase()}";
  }
```

This String extension capitalizes the first character of a String and returns it. This is very useful in showing usernames and other useful text in the application. If you can look at the code you can access the string in the function using this keyword. So basically this keyword gives you access to the String which you want to capitalize. You can use this extension in the code as follows -

```dart
Text('pranav masekar'.capitalizeFirstLetter()); // "Pranav masekar"
```

### Integer Extensions

```dart
extension IntExtensions on int {
  String formatAmount() {
    String amountString = toString();
    if (amountString.length <= 3) {
      return amountString;
    }
    String formattedAmount = '';
    int commaPosition = amountString.length % 3;

    if (commaPosition == 0) {
      commaPosition = 3;
    }

    formattedAmount += amountString.substring(0, commaPosition);

    while (commaPosition < amountString.length) {
      formattedAmount +=
          ',${amountString.substring(commaPosition, commaPosition + 3)}';
      commaPosition += 3;
    }

    return formattedAmount;
  }
}
```

This is the extension of integer value; it returns a String. This extension is helpful when we want to show the amount in the UI of the application. We want to add commas in the amount to create a more intuitive UI. So this extension does that for you. You can use this extension as follows -

```dart
int amount = 45000;
String formattedAmount = amount.formatAmount();
print(formattedAmount); // 45,000
```

### BuildContext Extensions

```dart
extension ContextExtensions on BuildContext {
  Size get screenSize {
    return MediaQuery.of(this).size;
  }

  double get screenHeight {
    return screenSize.height;
  }

  double get screenWidth {
    return screenSize.width;
  }
}
```

This context extension provides an easier way to access the current screen size like height and width. This is extremely helpful because we don't need to write the entire MediaQuer.of(context) to get the screen size. We can use this extension as this -

```dart
Size screenSize = context.getScreenSize();
```

### DateTime Extensions

```dart
extension DateFormatting on DateTime {
	
  String toDatewithoutSlashes() {
    final format = DateFormat('dd MMM yyyy');
    return format.format(this);
  }

  String toDateWithFirstMonth() {
    final format = DateFormat('MMM dd yyyy');
    return format.format(this);
  }

  String toDateWithSlashes() {
    final format = DateFormat('dd/MM/yyyy');
    return format.format(this);
  }
}
```

This is an extremely helpful extension to work with DateTime objects in Flutter. This requires the package called intl to be installed. This provides various ways to display dates in Flutter applications. We can use this as follows -

```dart
DateTime date = DateTime.now();
String formattedDate = date.toDateWithSlashes(); // 15/07/2023
```

**Congratulations** on finishing this blog on Flutter extensions! You have covered a lot of ground in this blog post. I encourage you to continue exploring Flutter extensions and to share your findings with the Flutter community.

Keep Fluttering 💙💙💙