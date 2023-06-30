---
title: "Shorebird: Empowering Flutter Developers with Seamless Code Updates"
datePublished: Fri Jun 30 2023 13:42:38 GMT+0000 (Coordinated Universal Time)
cuid: cljimjesd00050aihac3mgrr8
slug: shorebird
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1688132421796/f516d623-0c49-4d49-98c8-a483ead2edde.png
tags: tech, wemakedevs, codepush, shorebird

---

Welcome to our exciting blog dedicated to the latest advancements in Flutter technology. In this blog, we are going to explore the newest flutter tool for developers and businesses i.e. Shorebird. But before understanding what shorebird and hands-on demos let us first understand what is the problem with the current flutter development process. 

## Problems with current flutter - 

As we all know after building the application we need to publish it on the play store and app stores. These platforms take their time to approve the application according to their privacy policies. That is fine for larger updates like new features, but for small patch updates and bug fixes, we still need to go through this approval cycle again. This is time-consuming and may result in a business loss if there are any breaking changes in the production application. This exact problem is solved by Shorebird which provides CodePush for flutter.

## What is Shorebird CodePush?

**Shorebird CodePush** is a cloud service that allows developers to push app updates directly to users' devices. This is done without any app store approvals. You can explore the CodePush architecture [here](https://docs.shorebird.dev/architecture). Shorebird currently supports both Android and ios platforms and it is also free to use. You can explore the current development of CodePush on their [GitHub](https://github.com/shorebirdtech/shorebird/issues) and also on [Discord](https://discord.com/invite/shorebird).

## 🧑‍💻 Let’s dive into the demo - 

### **Step 1**: Installation of Shorebird CLI

For Linux/macos users run this command in the terminal - 

```bash
curl --proto '=https' --tlsv1.2 https://raw.githubusercontent.com/shorebirdtech/install/main/install.sh -sSf | bash
```

For Windows users run this command - 

```bash
powershell -exec bypass -c "(New-Object Net.WebClient).Proxy.Credentials=[Net.CredentialCache]::DefaultNetworkCredentials;iwr -UseBasicParsing 'https://raw.githubusercontent.com/shorebirdtech/install/main/install.ps1'|iex"
```

After the successful installation you can run - 

```bash
shorebird doctor
```

> Note: Currently shorebird supports the latest version of Flutter so to avoid any inconsistencies use the latest Flutter version.

### **Step 2:** Account creation 

Run the following command to create an account in Shorebird -

```bash
shorebird account create
```

Currently, shorebird uses Google OAuth2 to authenticate users.

If you already have an account you can use this -

```bash
shorebird login
```

### **Step 3:** Project setup 

Go to the root directory of your Flutter project ( I am using the default Flutter counter application ) and run this -

```bash
shorebird init
```

And provide your application name. 

This command does three things - 

* Created a unique app\_id for your application by which shorebird servers identify your application.
    
* Create a shorebird.yaml file which stores your app\_id.
    
* Add this shorebird.yaml file into pubspec.yaml file so that Shorebird is bound to your application assets.
    

> Note: This unique app\_id isn’t any kind of secret so you can safely store this in your GitHub repository.

You may get a warning that internet permission is required for the application. To fix that run -

```bash
shorebird doctor --fix
```

After successful installation and project setup, let’s run the application using the -

```bash
shorebird run
```

This is the same as the flutter run but Shorebird uses a fork of the flutter repository to add some Shorebird updaters.

**Step 3:** Building and releasing the application 

To release the application use the following command - 

```bash
shorebird release android
```

To get apk file use - 

```bash
shorebird build apk
```

Now install this released apk in your Android device and let’s do some application changes.

### **Step 4:** Update the application with CodePush

Create some changes to the application such as changing the text or adding some widgets. After changes run the patch command  -

```bash
shorebird patch android
```

After this command, terminate your running application on your device. Open it again, When the patch is applied, the first opening instance of the application downloads the latest patch and applies it on the next startup of the application. So kill the application again and open it. 

Now you will be able to see the changes in the application.

```bash
$ shorebird patch android
✓ Building patch (197.9s)
✓ Fetching apps (0.3s)
✓ Detected release version 1.0.0+1 (1.0s)
✓ Fetching release (51ms)
✓ Fetching Flutter revision (4ms)
✓ Fetching release artifacts (0.1s)
✓ Fetching aab artifact (44ms)
✓ Downloading release artifacts (1.8s)
✓ Creating artifacts (4.7s)

🚀 Ready to publish a new patch!

📱 App: shorebird_demo (6191bbac-423a-44d3-8fda-e28cff102462)
📦 Release Version: 1.0.0+1
📺 Channel: stable
🕹️  Platform: android [arm64 (48.91 KB), arm32 (34.62 KB), x86_64 (34.12 KB)]

✓ Creating patch (63ms)
✓ Uploading artifacts (0.7s)
✓ Fetching channels (43ms)
✓ Promoting patch to stable (45ms)

✅ Published Patch!
```

### **Step 5:** Github actions CI (Optional)

* To create a CI we need to generate a secret token for the application for that run
    
    ```bash
    shorebird login:ci
    ```
    
    After you sign in it will display the secret token and copy that token.
    
* Go to settings of your GitHub project -&gt; under Secrets and variables -&gt; Actions -&gt; New Repository Secret
    
    ```bash
    Key : SHOREBIRD_TOKEN
    Secret : <TOKEN>
    ```
    
*  Now create shorbird.yaml inside the .github/workflows folder and paste the code -
    
    ```yaml
    name: Shorebird Patch
    
    on:
      push:
        branches: [ "main" ]
    
    jobs:
      patch:
        defaults:
          run:
            shell: bash
    
        runs-on: ubuntu-latest
    
        steps:
          - name: 📚 Git Checkout
            uses: actions/checkout@v3
    
          - name: 🐦 Setup Shorebird
            uses: shorebirdtech/setup-shorebird@v0
    
          - name: 🚀 Shorebird Patch
            run: shorebird patch android --force
            env:
              SHOREBIRD_TOKEN: ${{ secrets.SHOREBIRD_TOKEN }}
    ```
    
*  Now push some changes into the repository to trigger CI.
    

### Upcoming Features of CodePush

* Support for native code changes
    
* Support for older Flutter version
    
* Support for assets
    
* Self-hosting of servers and many more.
    

To keep up with the development, join the [discord](https://discord.com/invite/shorebird) server and [github issues](https://github.com/shorebirdtech/shorebird/issues) page. 

That’s it we have successfully implemented CodePush solution into our flutter application. CodePush is still in an early stage so every feedback from developers matters. 

Keep Fluttering 💙💙💙