---
title: "Boost Your Flutter Development with Freezed"
datePublished: Sun Jul 23 2023 03:30:13 GMT+0000 (Coordinated Universal Time)
cuid: clkevsf5f000109l7drnugch7
slug: freezed
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1690015770563/c1497247-a606-4f38-b20f-2e7ee570d7b2.png
tags: serialization, flutter, flutter-examples, codegeneration, wemakedevs

---

# Introduction :

As our Flutter applications grow in complexity, it becomes tedious to manually define immutable model classes. We have to explicitly mark fields as final, implement copyWith methods, override == and hashCode, and more. This repetitive code adds friction during development. The Freezed package eliminates this busy work by auto-generating immutable model classes for us. Freezed leverages code generation to produce boilerplate-free immutable models based on simple abstract classes. It automatically handles copyWith, ==, hashCode and other utilities for our data classes. We can focus on the business logic while Freezed handles the immutable plumbing in the background.

In this blog, we'll see how Freezed can save us from writing repetitive code for immutable models. We'll look at how to define abstract classes and have concrete implementations generated for us. Freezed provides a clean syntax for declaring data classes without hand-writing tons of boilerplate. Let's see how Freezed can help us write less code while making our Flutter apps more robust!

# **Installing Packages :**

To get started with freezed we need to install the following packages -

```bash
flutter pub add freezed_annotation
flutter pub add json_annotation
flutter pub add --dev build_runner
flutter pub add --dev freezed
flutter pub add --dev json_serializable
```

We need freezed\_annotation and json\_annotation to mark our model classes so that build\_runner can create those for us automatically. Json\_serializable is used to generate toJson() and fromJson() methods.

# **Getting Started with Freezed Code Generation :**

Now let’s create our first data class with the help of freezed package. So create a file named address.dart and add this code  -

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690014877304/cbf4dbfd-94bb-4ab0-bb96-23b2042ce90f.png align="center")

Initially, your editor will be showing a lot of errors but don’t worry we will generate all those files shortly before that let's understand the code.

* First, we annotate our class with `@freezed` so that our class can be generated automatically.
    
* The `With` keyword defines the Address class and says it will be implemented in the generated \_$Address class.
    
* Then we define our variables inside the factory constructor.
    
* The fromJson factory constructor is used to generate toJson and fromJson methods for the data class.
    

Now go to the terminal in the root directory of the project and run this -

```bash
dart run build_runner build
```

With that, all the required files will be generated automatically.

# **Json Annotations and Default Values :**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690014999240/0e4621ae-4797-4f6e-bc98-0c63d17de3cc.png align="center")

* `@Default("Maharastra")` on the state property provides a default value of "Maharastra" if the state is not present in the JSON.
    
* `@JsonKey(name: "pin_code")` on the zipCode property controls the JSON key mapping. By default, Dart property names are snake\_cased when serialized to JSON. This annotation overrides that behaviour and maps zipCode to the JSON key "pin\_code" instead.
    

So in summary:

`@Default` - Provides default value if the property is missing from JSON

`@JsonKey` - Customizes JSON key naming for the property

# **Leveraging Freezed Copy Logic in Nested Classes :**

As of now, we have created a simple modal class that just stores the address but in real work scenarios, this address class will be a part of the user class. So let’s create a user class that contains the instance of the Address class.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690015050899/aa09f191-b104-4283-8bb0-5dcd2aee4466.png align="center")

After this run the build runner command to generate the code for the user class. With just that we created a nested class.

# Freezed vs Dart Data Class Generator Extension :

Most of us may have used a vs code extension i.e. dart data class generator. This extension does similar things to generate copyWith, toJson, and fromJson, overriding equality operators and hashcodes. But then why do we use the Freezed package 🤔 ?

Here are some differences where Freezed shines than an extension -

* ### Nested copyWith methods -
    
    In the previous example where we created a User class that has an instance of the Address class. Suppose we want to change the zip code of the address then if the classes are generated with a data class generator extension we have to do something like this -
    

```dart
user = user.copyWith(address: user.address.copyWith(zipCode: 411045));
```

But with the help of freezed it can be done like this -

```dart
user = user.copyWith.address(zipCode: 411045);
```

* ### Working with unions -
    
    Suppose we have a Payments model in the application which can be type UPI or PayPal or Bank. Without using the freezed class we would implement this scenario like this -
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690015317211/21ec07bf-e439-443b-ad4c-a9ea06a312ae.png align="center")
    
    But freezed can reduce this boilerplate by providing this -
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690015323366/289cc8bd-6a46-491b-b15f-db8e14ab2c96.png align="center")
    
    Here freezed creates those UPI, Bank and PayPal classes internally which reduces the boilerplate code.
    

We've reached the end of our journey learning how Freezed supercharges immutable models in Flutter! You should now understand the key benefits of using Freezed. Thank you for reading! Keep learning and creating amazing Flutter projects.

Keep Fluttering 💙💙💙