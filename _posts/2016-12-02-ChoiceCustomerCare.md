---
layout: post
title: "Building Xamarin.Forms apps in Azure with Choice Customer Care"
author: "Sarah Sexton"
author-link: "#"
#author-image: "{{ site.baseurl }}/images/authors/sarahse.jpg"
date: 2017-01-20
categories: [Mobile Application Development with Xamarin, Mobile DevOps]
color: "blue"
#image: "{{ site.baseurl }}/images/CCC.png" #should be ~350px tall
excerpt: Microsoft teamed up with Choice Customer Care to streamline the process of patients requesting help in hospital emergency rooms via a web-based API and a Xamarin.Forms cross-platform mobile application. This article describes the development process and the outcomes.
verticals: [Healthcare]
language: [English]
---

A team from Microsoft and Choice Customer Care LLC set out to create a system that would improve the process of hospital patients requesting help from nurses and doctors. 

In most hospitals today, patients press a call button when they need help, activating a light at the nurses station so that a healthcare worker can go find out what is needed. Sometimes that worker can fulfill the request; other times they must find a different worker to fulfill it. This system is inefficient and difficult to track.

Along with the initial architecture and version of the Choice Customer Care applications and web services, Microsoft assisted Choice Customer Care in creating a value stream map (VSM) to set a baseline for its software development lifecycle (SDL), in order to highlight existing and future DevOps best practices.

**Core team:** 

* Alicia Avril ([@AliciaSAvril](https://twitter.com/AliciaSAvril)) – Lead Developer, Choice Customer Care LLC
* Sarah Sexton ([@Saelia](https://twitter.com/Saelia)) – Technical Evangelist, Microsoft
* David Giard ([@DavidGiard](https://twitter.com/DavidGiard)) – Senior Technical Evangelist, Microsoft 
* Kevin Remde ([@KevinRemde](https://twitter.com/KevinRemde)) – Senior Technical Evangelist, Microsoft

**Other contributors:**

* Paul DeCarlo ([@pjdecarlo](https://twitter.com/pjdecarlo)) – Senior Technical Evangelist, Microsoft 
* Martin Schray ([@mschray](https://twitter.com/mschray)) – Senior Technical Evangelist, Microsoft
* Nick Landry ([@ActiveNick](https://twitter.com/ActiveNick)) – Senior Technical Evangelist, Microsoft
* Eric Rozell ([@EricRozell](https://twitter.com/EricRozell)) – Software Development Engineer, Microsoft
* Oren Novotny ([@Onovotny](https://twitter.com/onovotny)) – Principal Architect at BlueMetal, Xamarin Most Valuable Professional (MVP)

<img src="{{ site.baseurl }}/images/CCCphotos2.png" width="500">


We helped build a cross-platform solution that allows hospital patients to use a tablet app to specify what kind of help they need. This then routes that request directly to the appropriate healthcare worker's smartphone via a mobile app. Notifications are sent in real time, and response times are monitored and reported accurately with this system. 

We built the tablet and mobile client apps using [Xamarin.Forms](https://www.xamarin.com/forms), [Visual Studio 2015](https://www.visualstudio.com), and [Visual Studio Team Services (VSTS)](https://azure.microsoft.com/en-us/services/visual-studio-team-services/). The server was built on [Microsoft Azure](https://www.azure.com), with an [Azure SQL Database](https://azure.microsoft.com/en-us/services/sql-database/?b=16.50) and an exposed [Web App](https://azure.microsoft.com/en-us/services/app-service/web/) written as a web API. 

The rules for routing patient requests are held in a SQL Server database to track business logic and requests, using [Entity Framework](https://www.asp.net/entity-framework) to more easily work with the data. Real-time [push notifications](https://docs.microsoft.com/en-us/azure/notification-hubs/) are sent using [Azure Notification Hubs](https://azure.microsoft.com/en-us/services/notification-hubs/) so that healthcare providers can receive, accept, or reject requests immediately. Each new app build can be released through the [HockeyApp](https://www.hockeyapp.net/) beta distribution mechanism. Healthcare providers are assigned to beds, not patients, to avoid exposing health identifying information. 

## Customer profile

[Choice Customer Care](http://www.choicecustomercare.com) (CCC) is a Chicago-based, family-owned and operated software development shop putting the patient in the center of their hospital experience. With no revenue and no outside funding, CCC has been bootstrapping a minimum viable product of its system to improve communication and service for hospital patients. 

The Chief Executive Officer (CEO), Mark Mitchell, is an emergency room physician who was inspired to create CCC after feeling that the current hospital environment fails to meet the standards patients should expect for their care. Before seeking help from Microsoft, all aspects of development were handled personally by Mark, with design and finances handled by his wife, Laura Mitchell. Their daughter Alicia Avril handled all software development engineering of the project with the assistance of hired consultant Walid Johnson.

They have been working to finalize the system design so that with some assistance from Microsoft, CCC will be ready for a pilot run in an emergency room setting. The company currently has a pilot scheduled at Hospital Sisters Health System (HSHS) St. Francis Hospital in Effingham, Illinois. The pilot begins January 17, 2017, and will run for at least two months (to March 17), with the option to extend up to two more months if both the hospital and the company agree to do so.

<img src="{{ site.baseurl }}/images/CCCtweet0.png" width="500">


## Problem statement

If a hospital patient presses a nurse call button and waits more than 30 minutes for help in going to the bathroom, that frustration may be the only memory that sticks out to the patient. Hospital revenue is affected by negative reviews. 

- Hospital reimbursement from the government is growing more and more dependent on quality measures, including statistics such as patient satisfaction. 
- Medicare and Medicaid hold a pool of money for reimbursement to hospitals. Hospitals with high-ranking patient satisfaction scores get more reimbursement money from the government than low-ranking hospitals.
- One of the biggest issues in patient satisfaction currently, according to [PressGaney](http://www.pressganey.com) and other major surveys, is that patients don't feel as though their needs are being met. 

>*"I came up with the idea for my company when it came to my attention that a little old lady under my staff's care had wet herself in bed because her requests for bathroom assistance had not been answered for over half an hour." - Mark Mitchell, CEO*

## Solutions, steps, and delivery 

**Project scope**

Choice Customer Care offers a complete Wi-Fi-based system replacing the call button with a bedside tablet, and the flashing light at the nurses station with a modern mobile solution in every nurse's pocket.

The Choice Customer Care application consists of three components: 

* A tablet application affixed to a hospital bed, accessed by the patient in that bed.
* A cloud application consisting of a back-end SQL Server database, a REST web service exposed to client applications, a set of business rules, and a routing engine that sends push notifications to the phones of healthcare workers.
* A phone application that allows healthcare workers to receive patient requests and either accept or reject them. 

The diagram below shows the components working together.

![Architecture Diagram]({{ site.baseurl }}/images/CCCArchDiagram.png)


Choice Customer Care does not plan to use the App Store to distribute the tablet and phone applications to the devices. By opening up distribution of internal builds to beta testers through HockeyApp's beta distribution mechanism, CCC can scale testing to beta users regardless of location. HockeyApp also allows them to bypass slower official beta distribution channels provided by the Android and iOS stores. 

**Azure cloud application**

The cloud application exposes REST web services, allowing patients to send a request (via the tablet app) and healthcare workers to
accept or decline a request. 

The REST service calls a set of business rules to determine to which healthcare worker a request should be sent. The business rules and the current state of each worker and each request are persisted in an Azure SQL database that the web service reads from and writes to. 

We used Entity Framework to more easily work with the data in our Azure SQL database. Entity Framework is an object-relational mapper that enables .NET developers to work with relational data using domain-specific objects. It eliminates the need for most of the data-access code that developers usually need to write.

The cloud application notifies healthcare workers of patients' requests, via a message sent from Azure push notifications. 

**Tablet application**

The tablet application is built to be installed on an Android tablet that is affixed next to a patient’s bed. 

When patients require assistance, they press the icon corresponding to the type of assistance needed (for example, medical emergency, non-medical emergency, bathroom, water). Pressing the button calls a web service that routes the request based on the type of request and which healthcare worker is currently assigned to that bed. This sends a request notification to the mobile app of the worker responsible, according to the business logic. 

The tablet app broadcasts a push notification only to the specific worker responsible for that tablet. Workers must go to the patient's bedside to accept or deny the notification.

We created a new Xamarin.Forms Shared Project using the Model-View-ViewModel (MVVM) methodology. 

By associating the tablet to a bed, instead of a patient, we eliminated privacy concerns that could otherwise make this app much more difficult to develop and deploy.

Below is a screenshot of the main page of the tablet application:

![Screenshot Tablet 1]({{ site.baseurl }}/images/CCCScreenTablet1.png)


![Screenshot Tablet 2]({{ site.baseurl }}/images/CCCScreenTablet2.png)


**Phone application**

The phone application will be installed on each healthcare worker's smartphone. Workers can log into their shift, at which time they can assign themselves to the beds they are responsible for, or a charge nurse can assign them. The phone receives a push notification from 
the cloud application when a patient request is directed to the logged-in healthcare worker associated with this phone.

We created a new Xamarin.Forms Shared Project using the Model-View-ViewModel methodology. 

Below are some screenshots from the mobile application: 

<img src="{{ site.baseurl }}/images/CCCScreenPhone1.png" width="400">


<img src="{{ site.baseurl }}/images/CCCpush1.png" width="400">


**Xamarin, VSTS, and Visual Studio 2015**

The first step to the solution was to create a Xamarin.Forms application for Android and iOS. The CCC team designed the UX of the application to be simple and straightforward.

To lay out the buttons for the tablet and mobile applications, we created a responsive grid in Xamarin.Forms XAML. By specifying the image size and using the * operator on the end row and columns, the grid becomes centered on all screens. The * operator fills the remaining space on the screen after allocating space for the other rows/columns.

Xamarin.Forms allows you to customize the user interface beyond the built-in controls using [custom renderers](https://developer.xamarin.com/guides/xamarin-forms/custom-renderer/). There is also a custom component store that lets you use community-created controls quickly. In this case, we used the [Circle Image Control Plugin for Xamarin.Forms](https://github.com/jamesmontemagno/ImageCirclePlugin), which leverages custom renderers to create the circular buttons.
The team could then add images (including high-res images) to each platform-specific project, and Xamarin.Forms manages finding the appropriate image to display for each platform.

One lesson learned by the development team: In Visual Studio, always restore the project's NuGet packages from the Internet before beginning to work. Build or rebuild the solution, or right-click on your Solution explorer to restore NuGet packages. The first time, we were prompted to right-click ChoiceTabletApp.Droid to "Install Missing Feature(s)" and "Install Xamarin to build Android." In order to prepare one's development machine for the Xamarin environment, go to "Manage NuGet Packages for Solution," click the **Updates** tab, and scroll all the way down to Xamarin.Forms to check for updates.

![Xamarin.Forms NuGet Packages]({{ site.baseurl }}/images/CCCxam.png)


**Running apps on iOS and Android emulators**

Xamarin was chosen as the main technology for this project due to the ability to target multiple platforms (Android, iOS) with C#. 
A lot of great support is now available in the stable channel for both Xamarin Studio and Visual Studio with the latest release of Xamarin. 

The team at Xamarin has done an amazing job of documenting all of the requirements that you will need, so check out these pages for [iOS](https://developer.xamarin.com/releases/ios/xamarin.ios_10/xamarin.ios_10.0/#Requirements) and [Android](https://developer.xamarin.com/guides/android/platform_features/introduction-to-nougat/#Requirements). 
 
These instructions are for installing Android on a Windows 10 PC. To prepare a Mac for Xamarin development (8/iOS10/AndroidN), reference James Montemagno's documentation: [Preparing Machines for Xamarin Cycle](http://motzcod.es/post/150380059392/preparing-machines-for-xamarin-cycle).

* It was not possible to run any emulators using the Windows 10 Home Edition because it does not support Hyper-V. The solution we found for this problem was to upgrade to Windows Professional.
* When developing with a Surface Book or any other machine that uses a Skylake Intel processor, you must manually turn on your machine's processor compatibility for Hyper-V. 
   * [Solving issue with VS Android Emulator, Xamarin apps and Intel Skylake processor (i.e. when using a Microsoft Surface Book i7, Surface Pro 4, etc.)](https://blogs.msdn.microsoft.com/cesardelatorre/2016/02/06/solving-issue-with-vs-android-emulator-xamarin-apps-and-intel-skylake-processor-i-e-when-using-a-microsoft-surface-book-i7-surface-pro-4-etc/)
* Sometimes, Visual Studio 2015 hangs when checking whether an emulator is already running. The Output dialog suggests it detects that the emulator is already running, but it hangs at that step and does not deploy the app. When the emulator would freeze, we had to shut down the program manually and re-launch it. (This has been reported to the Visual Studio Product Team as "Bug 69902: Visual Studio 2015 Emulator for Android Freezing.")

**DevOps applied** 

![VSTS source code]({{ site.baseurl }}/images/CCCVSTS.png)


To begin our project, we needed a way to work on the code base remotely and securely with version control, work-item tracking, and continuous integration. Using Visual Studio Team Services, we were able to have everything we needed. After initial setup, we were able to invite team members into the project and begin collaboration.

VSTS simplified the use and integration of the following DevOps practices:

* **Continuous integration:** The code that we wrote and committed to VSTS (using the Git branching system) was integrated into a "build." The check-in of said code resulted in a "build" of our Web Service. See the screenshot below:

  ![CI build definitions]({{ site.baseurl }}/images/CCCVSTS3.PNG)
  

* **Continuous deployment:** We were able to publish a sample web page by deploying our Web API to a web app, at [https://choicecustomercarewebapi.azurewebsites.net/](https://choicecustomercarewebapi.azurewebsites.net/). See the screenshot of our VSTS environment below:

   ![CD VSTS editor]({{ site.baseurl }}/images/CCCVSTS5.PNG)

   * This uses Alicia's Azure subscription to deploy to a Web App service named ChoiceCustomerCareWebAPI in a staging slot. The Web API has both a staging and a deployment slot so that testing can be done using the staging slot before it is swapped out for the deployment slot and officially used in production. 

   ![CD Azure Website .net]({{ site.baseurl }}/images/CCCVSTS4.PNG)
   

As a new and small team, Choice Customer Care did not have any DevOps practices in place before the project began. CCC needed a way to easily update and maintain the back end with only one dedicated programmer on the team. This led to the team choosing the DevOps practice of continuous deployment so they could deploy to a web slot, and easily update the back-end code for the web API. We needed the back end online and accessible so that the tablet and mobile apps had a common point of communication that could access and update the database.

In the [**value stream mapping**](http://blog.gembaacademy.com/2008/02/24/lets-create-a-current-state-value-stream-map/) session, the team was able to map out the product lifecycle in order to better track existing DevOps best practices, as well as have a [visual representation](https://support.office.com/en-us/article/Create-a-value-stream-map-35A09801-999E-4BEB-AD4A-3235B3F0EAA3) of ongoing areas for improvement. The VSM identifies the 'Test' steps as the key areas of waste. This led to CCC distributing the app to HockeyApp as a plan for reducing manual testing.

![Value Stream Map]({{ site.baseurl }}/images/2017-01-06_CCC_VSM.JPG) 


There is currently no automated testing (written as AT on the diagram) being done on the tablet, mobile, or web service apps. Continuous deployment (CD) and release management (RM) are currently being employed to improve the delivery of the tablet and/or mobile apps to their respective devices.

For the pilot, CCC has a plan in place to distribute the mobile application via HockeyApp's Beta Distribution feature. Using HockeyApp, continuous integration is also implemented for the mobile app's Xamarin code via a VSTS build definition. Reference: [Build Your Xamarin App](https://www.visualstudio.com/en-us/docs/build/apps/mobile/xamarin). 

One lesson learned by the development team was to keep track of all passwords necessary for use in development technologies. Several passwords are used across the board; we had to keep track of tokens, secrets, numbers, usernames, and so on. One specific instance we encountered was when a team member generated a service in Azure without sharing the username and password with the rest of the team, and then went on vacation and couldn't be reached for several days. 

![David Giard, Sarah Sexton, Kevin Remde]({{ site.baseurl }}/images/CCCphotos1.png)


**Add HockeyApp analytics**

![HockeyApp Dashboard]({{ site.baseurl }}/images/CCCHockeyApp1.png)


We started off by following the HockeyApp documentation [How To Integrate HockeyApp with Xamarin](https://support.hockeyapp.net/kb/client-integration-cross-platform/how-to-integrate-hockeyapp-with-xamarin), which allowed us to easily set up crash reporting.

During development, we recorded the actual HockeyApp integration process on video to provide you with a rundown of exactly what you need to know to bring HockeyApp into your project!

In this video, we capture a real-life example of bringing HockeyApp into a Xamarin mobile app project, which includes:

* How to create a HockeyApp account and add team members to your app (See: Part 1 in [Video](https://channel9.msdn.com/Blogs/raw-tech/HockeyApp-in-XamarinForms)).
*	How to add HockeyApp crash analytics ([Android](https://components.xamarin.com/gettingstarted/hockeyappandroid)/[iOS](https://components.xamarin.com/gettingstarted/hockeyappios)).
*	How and why for adding HockeyApp custom event tracking in a *Xamarin.Forms* app (See: [Gavin Bauman’s Blog](http://theothergavin.net/hockey-app-for-xamarin-forms-no-problem/) and [this repo](https://github.com/toolboc/HockeyApp-Integration-for-Xamarin.Forms)).
*	How to integrate HockeyApp with VSTS to allow deploying builds directly to your user via HockeyApp ([Documentation](https://support.hockeyapp.net/kb/third-party-bug-trackers-services-and-webhooks/how-to-use-hockeyapp-with-visual-studio-team-services-vsts-or-team-foundation-server-tfs#installation-for-vsts)).
*	How to add the HockeyApp VSTS widget for quick access to HockeyApp analytics ([Documentation](https://support.hockeyapp.net/kb/third-party-bug-trackers-services-and-webhooks/how-to-use-hockeyapp-with-visual-studio-team-services-vsts-or-team-foundation-server-tfs#dashboard-widget-vsts-only-)).

The video is currently available on [Sarah Sexton’s Channel 9](https://channel9.msdn.com/Blogs/raw-tech/HockeyApp-in-XamarinForms).

Our project contained a scenario where we needed to capture custom metrics from our shared code in our Xamarin projects in order to assess user behavior. This required a customized solution using dependency injection to allow access to values in the shared code from our platform-specific projects. Here is a screenshot of the custom metrics that were added: 

![HockeyApp Custom metrics]({{ site.baseurl }}/images/CCCHockeyApp5.png)


We needed to use Dependency Injection to see this project through. Step-by-step instructions on how this was done are already available on [Gavin Bauman’s Blog](http://theothergavin.net/hockey-app-for-xamarin-forms-no-problem/) and [Paul DeCarlo's GitHub repo (HockeyApp Integration for Xamarin Forms)](https://github.com/toolboc/HockeyApp-Integration-for-Xamarin.Forms).

Here is a picture of HockeyApp tracking crash events over time in our Android tablet app: 

![HockeyApp Android Graphs]({{ site.baseurl }}/images/CCCHockeyApp2.png)


Here is a picture of HockeyApp crash reporting in action:

![Tablet Crash report]({{ site.baseurl }}/images/CCCcrash.png)


**Integrate HockeyApp into VSTS**

The [HockeyApp documentation](https://support.hockeyapp.net/kb/third-party-bug-trackers-services-and-webhooks/how-to-use-hockeyapp-with-visual-studio-team-services-vsts-or-team-foundation-server-tfs) outlined clear steps for integrating HockeyApp services with VSTS. Once set up, widgets can easily be added to an existing VSTS dashboard to allow quick navigation to analytics and reports. 

![VSTS Dashboard Integration]({{ site.baseurl }}/images/CCCHockeyApp4.PNG)


![VSTS Build Definition]({{ site.baseurl }}/images/CCCHockeyApp3.png)


Another lesson we learned during this project was that documentation online leaves out many of the required steps to connect VSTS to HockeyApp for beta distribution with Android.

**Build definitions for Xamarin Android apps in VSTS**

Here are some things you will need to know to connect VSTS to HockeyApp for beta distribution with Android:

- **Step 1.** Go through [How to use HockeyApp with Visual Studio Team Services (VSTS) or Team Foundation Server (TFS)](https://support.hockeyapp.net/kb/third-party-bug-trackers-services-and-webhooks/how-to-use-hockeyapp-with-visual-studio-team-services-vsts-or-team-foundation-server-tfs). This will link your HockeyApp instance to Visual Studio Team Services.
- **Step 2.**	For Android, you need to have a **signed** application to properly distribute to HockeyApp. This is critically important, and repeated again below to drive the point.

  To do this the easy way:
  
    1. Go to the following link and skip to "Archive for Publishing" (essentially just right-click your project and select "Archive"): [Part 1 - Preparing an Application for Release](https://developer.xamarin.com/guides/android/deployment,_testing,_and_metrics/publishing_an_application/part_1_-_preparing_an_application_for_release/). 
    
    2. Next, create a signing identity by following: [Part 2 - Signing the Android Application Package](https://developer.xamarin.com/guides/android/deployment,_testing,_and_metrics/publishing_an_application/part_2_-_signing_the_android_application_package/).

    (**Note:** If you are able to create a local archive for distribution, and it properly deploys to devices—not an emulator—through side-loading, you can validate that your locally built .apk is sanitized for building in VSTS. That is to say, if it works locally, you can make it work in VSTS.)

    After you have created a signing identity, it will be located at **C:\Users\USERNAME\AppData\Local\Xamarin\Mono for Android\alias\alias.keystore**

    (For additional tips on keeping the signing key secure, see [https://www.visualstudio.com/en-us/docs/build/apps/mobile/secure-certs](https://www.visualstudio.com/en-us/docs/build/apps/mobile/secure-certs).)
    
    In the example below, we used an alias of "test" with password "test123". The signing identity seems arbitrary, but is required to install to Android devices.

    Copy the contents of your signing identity to your project and check it in (required):

    ![Keystore]({{ site.baseurl }}/images/CCCkeystore.png)
    

- **Step 3.** Create a build definition in VSTS similar to the screenshots below. Reference: [Build Your Xamarin App](https://www.visualstudio.com/en-us/docs/build/apps/mobile/xamarin). 

  ![VSTS Build Definition for Mobile Code]({{ site.baseurl }}/images/CCCBuildVSTS.gif)
  

- **Step 4.** If you do not sign and Zipalign your APK, it will NOT install on devices. It *may* install on emulators, but will never install on a device. This is by design in the Android OS. 
- **Step 5.** Successful builds should now drop into HockeyApp.

  ![HockeyApp Builds]({{ site.baseurl }}/images/CCCHockeyAppBuild.PNG)


  Click a build to bring up download options:

    ![HockeyApp Download]({{ site.baseurl }}/images/CCCHockeyAppDownload1.PNG)
    

  Download to your device:

    ![HockeyApp Download Page]({{ site.baseurl }}/images/CCCHockeyAppDownload2.PNG)
    

- **Step 6.** On your test devices, go to [http://install.hockeyapp.net](http://install.hockeyapp.net) in your mobile browser to get the latest build right from your device. There is even an option to scan a QR code to open the download page on your phone. Enable unknown sources in your device's security settings to allow the installation of apps from sources other than the Google Play Store to do this.

**Azure push notifications**

After building out a basic skeleton of our Xamarin.Forms mobile app, we began work to implement push notifications by installing the plugin for Xamarin Components Push Notifications. 

We implemented a back end for the push notifications by configuring an [Azure Notification Hub](https://azure.microsoft.com/en-us/documentation/articles/xamarin-notification-hubs-push-notifications-android-gcm/) and enabling [Firebase Cloud Messaging (FCM)](https://firebase.google.com/docs/android/setup). After that, we updated our server project to send push notifications, which must be done natively for each platform. We configured Google authentication for [Android](https://azure.microsoft.com/en-gb/documentation/articles/app-service-mobile-how-to-configure-google-authentication/) first. 

Visual Studio Emulators for Android do not work with push notifications, but [this tutorial](https://www.youtube.com/watch?v=AfWqwN2kcZ0) allows you to install the Google Play services GAPPS. (Note that this tutorial will not work for Android 6.0.0 Marshmallow as of the time of this writing, but it does work for 5.1.) Continuing with the video tutorial, we created a package name in our Android manifest, and found the keytool to collect our Android debug key and store pass. After this, we were able to configure and run the Android project. The Xamarin component for push notifications automatically integrates with Azure Notification Hubs Server. 

## General lessons 

![Some photos from the Hackfest]({{ site.baseurl }}/images/CCCphotos.png)


### Entity Framework

Implementing many-to-many relationships in Entity Framework proved to be a challenge. On multiple occasions, linking objects in the webAPI would cause a copy of an already existing object to be added to the database. Through testing and research, we found that this is an issue common to Entity Framework that can be alleviated either by attaching the already existing object to the database, which had mixed results, or altering the method of discovering the already existing object. In our case, changing a passed object reference to a renewed call to Find() using an ID solved the issue.

![Whiteboard]({{ site.baseurl }}/images/CCCWhiteboard.png)


### Scoping

Keeping the scope narrowly focused was a problem, given that several loosely coupled pieces (tablet app, web service, SQL, phone app) interacted with one another.

### Visual Studio 2015 Intellisense

Intellisense in Xamarin.Forms code does not work in Visual Studio 2015. One workaround for this issue is to install third-party software such as [ReSharper](https://www.jetbrains.com/resharper/download/). 

### VSTS permissions

Martin Schray and Kevin Remde helped us with DevOps, but both of them needed special permissions in VSTS that took us a while to find and grant to them. The trick to doing this is to click the **Users** tab next to Home, which will bring you to a *"_user"* page, such as at *https://choicecare.visualstudio.com/_user.* Note the screenshot below:

![VSTS User Permissions]({{ site.baseurl }}/images/CCCVSTS2.png) 


### Emulators

When developing and testing our mobile app, sometimes it would crash unexpectedly. We only encountered this issue in the Android emulator. A workaround we found for this was to uninstall the app on the Android emulator device before relaunching. Simply rebuilding and deploying did not work.

## Conclusion

### Our impact

- Microsoft's engagement with Choice Customer Care has resulted in a tablet app, a mobile app, and a Web API connected to an Azure SQL database.
- The Xamarin cross-platform implementation will allow Choice Customer Care to begin beta testing with a real emergency room in Hospital Sisters Health System (HSHS) St. Francis Hospital, in Effingham, Illinois, on January 17, 2017. 
- Adding DevOps best practices for their initial work, as well as using the value stream map (VSM) to identify future areas for improvement in their development lifecycle, will give Choice Customer Care not only a good idea of where they are, but where they may want to go to better improve their SDL efficiencies.
- Expected impact from this project will be improved patient satisfaction scores in hospital partners, which will be measured during the beta in January, using hospital consumer assessments of healthcare providers and systems, surveys such as [www.pressganey.com](http://www.pressganey.com) and [www.hcahpsonline.org/home.aspx](http://www.hcahpsonline.org/home.aspx), as well as distributing their own surveys before, during, and after the beta for more immediate results. 

### Opportunities going forward

- **Authentication.** One of the next steps for the app is to provide authentication, which will allow storing unique user state and profiles for the healthcare workers using the app. CCC plans to implement authentication using [Azure Active Directory (AAD) Business to Consumer (B2C)](https://github.com/Azure-Samples/active-directory-b2c-xamarin-native/) by integrating it into the Xamarin.Forms app using the Microsoft Authentication Library (MSAL), which is available as a NuGet package. 
- **HIPAA compliance.** In the healthcare industry, privacy and confidentiality are crucial. Choice Customer Care is built for healthcare professionals dealing with patient information, so it is critical that its solution adheres to the Health Insurance Portability and Accountability Act (HIPAA). We built an implementation that is using Azure services that are in scope for HIPAA and the HITECH Act through the Microsoft Business Associate Agreement. For more information, read the Microsoft Trust Center document on [HIPAA](https://www.microsoft.com/en-us/TrustCenter/Compliance/HIPAA).
- **Scale.** This solution was designed to support an industry with thousands of locations around the globe. In order to match this addressable market, it's important that the back-end solution can scale to meet the demand as more users and hospitals come online. We selected Azure services that with dynamic load-scaling features, such as [Azure SQL](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-scale-on-the-fly), can scale up or down based on workload. Going forward, CCC is building an implementation test plan that includes these services. 
 

<img src="{{ site.baseurl }}/images/CCCtweet10.png" width="600">


* [Choice Customer Care website](http://choicecustomercare.com)
* Complete demo video:

![Video Image]({{ site.baseurl }}/images/CCCDemoVideo.gif) 


## Additional resources

A list of links to resources that complement our story:

- Documentation

  - [Azure Notification Hubs Notify Users with .NET backend](https://docs.microsoft.com/en-us/azure/notification-hubs/notification-hubs-aspnet-backend-windows-dotnet-wns-notification)
  - [Xamarin.Forms Modal Pages](https://developer.xamarin.com/guides/xamarin-forms/user-interface/navigation/modal/)
  - [Enhanced User Notifications](https://developer.xamarin.com/guides/ios/platform_features/introduction-to-ios10/user-notifications/enhanced-user-notifications/) 
  - [How to use HockeyApp with Visual Studio Team Services (VSTS)](https://support.hockeyapp.net/kb/third-party-bug-trackers-services-and-webhooks/how-to-use-hockeyapp-with-visual-studio-team-services-vsts-or-team-foundation-server-tfs#dashboard-widget-vsts-only-)
  - [Authenticating Users with Azure Mobile Apps](https://developer.xamarin.com/guides/xamarin-forms/web-services/authentication/azure/)
  - [Build Your Xamarin App](https://www.visualstudio.com/en-us/docs/build/apps/mobile/xamarin) 

- Blog posts

  - [Interactive Notifications with Notification Actions](http://www.thinkandbuild.it/interactive-notifications-with-notification-actions/) 
  - [Enhanced Notifications in Android N with Direct Reply](https://blog.xamarin.com/enhanced-notifications-in-android-n-with-direct-reply/)
  - [Gavin Bauman: "HockeyApp for Xamarin.Forms? No Problem!"](http://theothergavin.net/hockey-app-for-xamarin-forms-no-problem/)
  - [Sarah Sexton's Video: HockeyApp in Xamarin.Forms](https://channel9.msdn.com/Blogs/raw-tech/HockeyApp-in-XamarinForms)

- GitHub repos

  - [Xamarin.Forms Dev-Days Slide Deck, Hands-On Labs, & Documentation](https://github.com/xamarin/dev-days-labs/blob/master/SlideDecks/Dev%20Days%202%20-%20Xamarin.Forms.pdf) 
  - [Integrate Azure AD B2C into a Xamarin forms app using MSAL](https://github.com/Azure-Samples/active-directory-b2c-xamarin-native/)
  - [Eric Rozell's Xamarin.Forms Simple Page Navigation Example](https://github.com/rozele/XamarinFormsNavigationExample) 
  - [Sarah Sexton's Xamarin.Forms Simple Phone App Example](https://github.com/SarahSexton/Xamarin-Phone)
  - [Paul DeCarlo's GitHub repo (HockeyApp Integration for Xamarin Forms](https://github.com/toolboc/HockeyApp-Integration-for-Xamarin.Forms)
