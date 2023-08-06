---
title: "Sentry - The Secret Weapon for Flutter Error Tracking"
datePublished: Sun Aug 06 2023 03:30:10 GMT+0000 (Coordinated Universal Time)
cuid: clkyvyacp00020ajueb6ch7kv
slug: sentry
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1691215121473/5085367e-6ec3-4b35-8e65-a124fe5d120e.png
tags: flutter, error-tracking, sentry, wemakedevs, release-management

---

# **Introduction :**

No one wants their flawless app to crash when it hits the hands of users. But the reality is that even the most robust Flutter apps can fail in production due to unanticipated errors and bugs. Without a proper error monitoring system, these crashes could go undetected, leading to frustrated users and bad reviews. This is where Sentry comes in. Sentry is an error-tracking tool that helps developers monitor and fix crashes in real time. By integrating Sentry into your Flutter workflow, you get notified when errors occur in your live app. The detailed crash reports and debugging tools provided by Sentry enable you to quickly resolve issues before they impact your users. With Sentry, you can say goodbye to nasty production errors and focus on building five-star Flutter apps. In this post, we'll dive into how to properly implement Sentry logging and tracking in a Flutter app to revolutionize your error handling.

# **What is Sentry?**

Sentry is an open-source error monitoring and logging platform that allows developers to identify and quickly resolve errors and exceptions in real time. The key capabilities of Sentry include:

* Detailed crash reporting - Sentry provides full stack traces and context for crashes and errors to help pinpoint their root cause.
    
* Real-time alerting - Sentry sends notifications through email, Slack, GitHub, etc when errors occur so developers can act quickly.
    
* Integration with multiple platforms - Sentry integrates seamlessly with popular frameworks like React, Flutter, Node.js, Python, PHP, .NET, etc.
    
* Powerful data visualization - Useful dashboards, graphs and analytics provide insight into error trends and frequency.
    
* Robust security and compliance - Sentry meets the highest security standards like SOC 2 and GDPR.
    

# Sentry Setup :

Before actually going to deal with flutter and dart we need to create and set up a project on the sentry website. So open the sentry website by following this link -

[https://sentry.io/welcome/](https://sentry.io/welcome/)

and create a free account using available options like Google and GitHub. 

After your account is created you will see a dashboard something like this - 

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691214326161/ca36c200-5e78-44a4-a66d-c031a4fa9147.png align="center")

Now go to the projects tab and click on create project - 

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691214348721/30edac97-430f-4fc4-ac1b-403d59dc99d1.png align="center")

Select Flutter and give a name to the project -

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691214406570/9ce89172-17d1-4e10-bfd8-f02cc693ece6.png align="center")

So now our project is created on Sentry, now let’s connect this project with our flutter application. So open your flutter project, for this blog, I am going to use a music application you can choose any project.

# Flutter Setup :

We need to install the official Sentry SDK to start using Sentry in our project. So in pubspec.yaml add this package - 

```yaml
sentry_flutter: ^7.9.0
```

Now head towards the main.dart file and add these configuration lines -

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691214489104/b0d1263b-7243-47d2-ad4a-ce4a20df4ed2.png align="center")

* SentryFlutter.init() function initializes Sentry for the Flutter app asynchronously and passes in a callback to configure the options.
    
* options.dsn sets the DSN (Data Source Name) which is like the address for your Sentry project that the errors will be sent to. This will be different for every flutter project on the sentry platform.
    
* tracesSampleRate configures sampling rate for performance monitoring. 0.5 means 50% of traces will be captured. This is set low to avoid overuse of the free plan of sentry. 
    
* tracesSampler sets a custom sampler function that also returns 0.5 to sample 50% of transactions for performance monitoring.
    

# **Catch Error in Sentry :**

Here’s how we catch error in the sentry using a try-catch block and reports it to the sentry dashboard - 

```dart
FutureEither<Response> getSongs() async {
    try {
      String url =
          "https://api.musixmatch.com/ws/1.1/chart.tracks.get?apikey=<API-KEY>";
      final Response response = await dio.get(url);
      if (response.statusCode == 200)
        return right(response);
      else
        throw "Something went wrong";
    } catch (exception, stackTrace) {
      await Sentry.captureException(
        exception,
        stackTrace: stackTrace,
      );
      return left(exception.toString());
    }
  }
```

Now to test our sentry we can change the URL of the API so that we get an error and eventually reported it in the sentry.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691214604997/9c6356ef-df9b-4de6-92c9-5ba8c4372f6f.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691214626400/c09ddb93-8065-4925-8ee7-9467cca54d1c.png align="center")

# Sentry in release mode only :

Many times we want to use sentry-like error monitoring tools only in production releases of the application so for that we can change the main.dart file as follows -

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691214680146/878c3313-6cc1-4b9c-8f95-4be831901f60.png align="center")

**Congratulations** - you've now successfully integrated Sentry into your Flutter app for robust error and performance monitoring! With Sentry, you can rest assured knowing that any errors or crashes in your production app will be promptly detected and debugged. Leverage the detailed reports and data from Sentry to continuously improve your app's code quality. As your Flutter skills grow, integrate Sentry into all your projects to level up your error-handling abilities. Exceptional error monitoring is just a few simple steps away! Focus on building those flawless 5-star apps while Sentry watches your back.

Keep Fluttering 💙💙💙