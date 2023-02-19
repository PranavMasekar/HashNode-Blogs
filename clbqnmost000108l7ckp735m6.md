# Flutter Equatable

Hello everyone, in this article we are going in-depth with the Flutter Equatable package. We will understand why we use this package. What does it provide? What problem does it solve? and many more.

So let’s first understand what is the problem statement here

So assume you have a simple class i.e. MyClass which holds one integer value index. We created two objects of this class with an index value of 1, and we compared these two objects using the “==” operator.

```dart
class MyClass{
  int index;
  MyClass({required this.index});
}

void main(){
  MyClass obj1 = MyClass(index:1);
  MyClass obj2 = MyClass(index:1);
  print(obj1==obj2);
}
```

As two objects have the same index value we expect true in the result but in reality, it gives the result as false.

So what’s happening here ??

To understand this we need to know about how dart works with objects. Dart compares objects with their memory address, not with the content/property of objects.

So the two objects we created have different memory addresses so it results in false.

We understand what’s happening with objects but practically we need to compare the objects with their content values i.e. index in our case.

To solve this problem we have two ways:-

*   By overriding existing “==” operator 
    
*   Using Equatable Package
    

Overriding the “==” operator creates unwanted code in the class and may be very complicated for bigger classes.

So we use the Equatable package,

To use the Equatable package add 

```yaml
dependencies:
  equatable: ^2.0.5
```

We need to edit our existing classes a little bit to get our code working -

```dart
import 'package:equatable/equatable.dart';

class MyClass extends Equatable {
  final int index;
  MyClass({required this.index});
  @override
  List<Object> get props => [index];
}

void main() {
  MyClass obj1 = MyClass(index: 1);
  MyClass obj2 = MyClass(index: 1);
  print(obj1 == obj2);
}
```

First, we will extend our class with the Equatable class to access the functions.

Now we override the existing get props method from the Equatable package 

We list down the variable which we need to use while comparing the objects in our its just one variable i.e. index.

And that’s it just extending the class and overriding one method will do our job. Running the above program gives true results.

So what does Equatable actually do ???

The Equatable package takes the variables in List i.e. props and considers only those variables while comparing objects in dart.

Usage : 

We use this Equatable package heavily in the Flutter BLoC pattern where we continuously release new states. We compare the current state with the previous state to decide whether a widget tree should be rebuilt or not.

  
I hope this blog will provide you with sufficient information on Trying up the **Equatable** package in your flutter projects. We showed you what the Equatable package is. its properties of the Equatable package. We made a demo program for the Equatable package. So please try it.