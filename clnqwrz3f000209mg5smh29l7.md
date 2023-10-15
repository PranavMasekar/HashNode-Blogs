---
title: "Brushing Up Your Flutter Skills: Custom Painting Essentials"
datePublished: Sun Oct 15 2023 03:30:13 GMT+0000 (Coordinated Universal Time)
cuid: clnqwrz3f000209mg5smh29l7
slug: custom-painter
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1697291603296/bcc72e07-3e18-4f68-9e71-568e8eabaac8.png
tags: user-interface, user-experience, design, flutter, wemakedevs

---

# Introduction

Welcome to the colourful world of Flutter custom painting! In this guide, we will embark on an exciting journey to unlock the true potential of Flutter's CustomPainter. If you've ever wanted to create unique, visually stunning, and interactive elements in your Flutter apps, you're in the right place. Flutter, with its versatile toolkit, caters to many UI requirements, but there comes a point where you need to break free from the constraints of standard widgets. This is where the magic of Custom Painter in Flutter unfolds. Custom Painter is not just an option; it's a necessity for those seeking to craft one-of-a-kind, visually captivating, and interactive user interfaces.

# Custom Painter

At its core, a Custom Painter in Flutter is a canvas on which you, as a developer, become an artist. It's a versatile tool that allows you to draw, paint, and craft your user interface elements. Think of it as a blank sheet of digital paper, ready for your creative strokes. With Custom Painter, you have complete control over how things look, from simple shapes and lines to intricate graphics. Custom Painter empowers you to design and draw UI elements that standard widgets can't match. It's a must-know skill for any Flutter developer looking to elevate their app's visual appeal and interactivity.

# Canvas

An Offset represents a 2D point in space, usually expressed in terms of coordinates, where (0, 0) typically represents the top-left corner of the canvas. Flutter uses a Cartesian coordinate system, where the top-left corner of the canvas is (0, 0), and the x-axis extends horizontally to the right, while the y-axis extends vertically downward. Positive x-values move to the right, and positive y-values move downward. Offset can also be used to move a point or a shape on the canvas. By applying an Offset to an existing point, you can shift it to a new location.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697289878178/d1e5977c-b2d1-40c6-b6a1-9732124071a6.png align="center")

# CustomPaint Widget

The CustomPaint widget in Flutter is a versatile container for custom graphics and drawings. It takes a CustomPainter as a parameter, allowing developers to define their rendering logic. With CustomPaint, you can create complex custom UI elements, charts, diagrams, or any visual content by painting directly on the canvas, giving you full control over the appearance and interactivity of your app's components.

# Rectangle

```dart
class MyCustomPainter extends StatelessWidget {
  const MyCustomPainter({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text("Flutter Custom Painter"),
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
      ),
      body: Container(
        width: MediaQuery.of(context).size.width,
        height: MediaQuery.of(context).size.height,
        padding: const EdgeInsets.all(10),
        child: CustomPaint(
          painter: QuadrantPainter(),
        ),
      ),
    );
  }
}

class ShapePainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()
      ..color = Colors.blue
      ..style = PaintingStyle.fill;

    final center = Offset(size.width / 2, size.height / 2);
    const rectWidth = 100.0;
    const rectHeight = 100.0;

    final rect = Rect.fromCenter(
      center: center,
      width: rectWidth,
      height: rectHeight,
    );

    final path = Path()..addRect(rect);

    canvas.drawPath(path, paint);
  }

  @override
  bool shouldRepaint(CustomPainter oldDelegate) {
    return false;
  }
}
```

**PaintingStyle:** PaintingStyle is an enum in Flutter used to define how shapes or paths are painted on the canvas. It has two values: `PaintingStyle.fill` and `PaintingStyle.stroke`

**Rect.fromCenter():** Rect.fromCenter is a constructor in Flutter that creates a rectangle with a specified width and height, centered at a given Offset point.

**Path():** Path is a class in Flutter used to define a series of drawing commands for custom shapes, lines, and curves.

**canvas.drawPath():** canvas.drawPath is a method in Flutter's Canvas class. It is used to render the specified path onto the canvas. This is how you draw custom shapes or paths defined using the Path class.

**shouldRepaint():** This method determines whether the custom painter should repaint or not. It takes an old delegate and returns a boolean. If it returns true, the painter is repainted; if false, it is not.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697290221413/d0834154-4256-42e1-8209-10f302b05908.jpeg align="center")

# Circle

```dart
class ShapePainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()
      ..color = Colors.blue
      ..style = PaintingStyle.stroke
      ..strokeWidth = 2.0;

    final center = Offset(size.width / 2, size.height / 2);
    const radius = 50.0;

    canvas.drawCircle(center, radius, paint);
  }

  @override
  bool shouldRepaint(CustomPainter oldDelegate) {
    return false;
  }
}
```

The .drawCircle method is used in Flutter's custom painting to draw a circle on the canvas. It takes three parameters: centre, radius and a paint object.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697290455051/f5794332-598e-4292-92f5-fd7ef461f152.jpeg align="center")

# Quadrant

```dart
class QuadrantPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    const gradient = LinearGradient(
      colors: [Color(0xffBA5370), Color(0xffF4E2D8)],
      begin: Alignment.bottomCenter,
      end: Alignment.topRight,
    );

    Paint paint = Paint()
      ..color = Colors.blue
      ..shader =
          gradient.createShader(Rect.fromLTWH(0, 0, size.width, size.height))
      ..style = PaintingStyle.fill;
    Offset center = const Offset(0, 0);
    double radius = size.width / 2;

    Path path = Path();
    path.moveTo(center.dx, center.dy);
    path.lineTo(center.dx + radius, center.dy);
    path.lineTo(center.dx, center.dy + radius);
    path.arcTo(
      Rect.fromCircle(center: center, radius: size.width / 2),
      0,
      pi / 2,
      true,
    );
    path.close();
    canvas.drawPath(path, paint);
  }

  @override
  bool shouldRepaint(CustomPainter oldDelegate) => false;
}
```

**moveTo:** `.moveTo(dx, dy)` is a method in the Path class that is used to set the starting point of a path. It moves the "pen" to the specified coordinates (dx, dy) without drawing a line.

**lineTo:** `.lineTo(dx, dy)`is another method in the Path class. It draws a straight line from the current point (where the "pen" is) to the specified coordinates (dx, dy). It creates a line segment between the current point and the new point.

**arcTo:** .arcTo(Rect rect, double startAngle, double sweepAngle, bool forceMoveTo) is a method in the Path class used to add an arc to the path. It's particularly useful for drawing parts of circles or curved shapes.

**Rect rect:** The bounding rectangle that defines the position and size of the arc. double startAngle: The starting angle of the arc in radians. double sweepAngle: The angle to sweep in radians (positive for clockwise, negative for counterclockwise).

**bool forceMoveTo**: If set to true, it moves the "pen" to the starting point of the arc without drawing a line, ensuring that the arc is not connected to the previous path.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697290469236/0bd5793d-56dd-4b89-b261-1da378fa6586.jpeg align="center")

**Congratulations** on reaching the end of this blog! I encourage you to keep experimenting with Custom Painter in Flutter. The world of custom graphics and visual storytelling is at your fingertips. Embrace your creativity, explore, and craft unique user interfaces that captivate your audience.

Happy Fluttering 💙💙💙