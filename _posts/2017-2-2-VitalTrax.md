---
layout: post
title:  Xamarin.Forms application, "Wing Enterprise"
author: DaveVoyles
author-link: http://www.DaveVoyles.com
#author-image: {{ site.baseurl }}/images/davevoyles-headshot.png
date:   2017-01-20
categories: [Mobile Application Development with Xamarin]
color: "blue"
#image: {{ site.baseurl }}/images/screens/vitaltrax-logo.png
excerpt: VitalTrax has delivered a platform for healthcare administrators to offer clinical trials in a modern way.
language:  [English]
verticals: [Healthcare]
---

From Tuesday, January 17th through Thursday the 19th, Microsoft Technical Evangelists worked on-site,
as well as remotely in Denver and New York City, to build a proof of concept mobile application 
and improved DevOps experience with VitalTrax, a healthcare startup based out of Philadelphia. 

For the mobile solution, Xamarin.Forms, a cross-platform C# platform was used to create applications 
on iOS, Android, and Windows, with most of the codebase living within a single project. 

We also walked the VitalTrax leadership through a Value Stream Mapping (VSM) process, to map out
their existing software lifecycle in order to allow them to better understand their process as well 
as note areas for improvement through the intelligent application of DevOps best practices.
 

**Key technologies used:**  

* Xamarin
* Visual Studio Team Services
* HockeyApp
* Azure App Services


 **Core Team:**

  * [Andy Reitano](http:/www.twitter.com/BatslyAdams) - Technical Evangelist, MSFT
  * [Adina Shanholtz](http:/www.twitter.com/FeyTechnologist) - Technical Evangelist, MSFT
  * [Jerry Nixon](http:/www.twitter.com/JerryNixon) - Technical Evangelist, MSFT
  * [Kevin Remde](http:/www.twitter.com/kevinremde) - Technical Evangelist, MSFT
  * [Dave Voyles](http:/www.twitter.com/DaveVoyles) - Technical Evangelist, MSFT


 
## Customer profile ##
- [VitalTrax](http://vitaltrax.net/)

- Philadelphia, PA

VitalTrax is comprised of remote and on-site employees, who work out of ic3401, a Philadelphia incubator
run by the University City Science Center, whom Microsoft has partnered with to create the Philadelphia
Reactor. 

 
## Problem statement ##

The company already had a web based solution for healthcare companies to create and administer
trials to patients, but were looking for a native experience on the mobile front, in an effort to 
differentiate their product from the rest of the market and drive platform adoption. 

At the moment, healthcare companies can generate surveys and trials from their web portal, which was
built on Microsoft’s open source .NET core. Patients then log into the portal through a browser
and fill out the forms, but VitalTrax was interested in taking things to the next level. 

CTO Todd Kueny envisioned the ability to notify users when a new trial had appeared, or perhaps 
when one was nearing a deadline for completion the user needed to be informed.  With push notifications,
this can happen. 

Moreover, a dynamic UI was necessary for each healthcare company administering trials. 
The fonts, colors, and images would need to be unique to each company. By dynamically
generating markup (XAML) with C# and Xamarin, we can do this. Simply read a JSON file from the
server, parse the key / value pairs, and widgets can be generated!

 
## Solution, steps, and delivery ##

### Xamarin ###

Our solution here had fulfill four requirements:

1.	Push notifications to alert the patient when a trial needed to be filled
2.	Offline sync / Local storage to store data on the device when connectivity is limited
3.	Dynamically generate UI based on a JSON file from the server
4.	Allow for web-based trials to still be accessed within the application

Xamarin is a cross-platform C# framework for creating applications. It isn’t limited to
mobile devices either, as it can also export to Microsoft’s Universal Windows Platform (UWP),
as well as Apple’s Mac operating system, OS X.

Perhaps the greatest benefit to Xamarin is the large code re-use that it allows for.
Until recently, developers needed create unique applications for each mobile platform, and 
write those apps in the platform’s native language. In the case of iOS, it would be Objective-C or Swift,
Java or C++ for Android, and C# or C++ for Windows Mobile. 

  ![Image of solution]({{site.baseurl}}/images/screens/VT-Project.png)

Even better, Xamarin.Forms allows developers to write platform specific code in its respective
project, but most of your UI can be shared within a single set of files. Forms is a new library
that enables you to build native UIs for iOS, Android and Windows Phone from a single, shared C# codebase.
It provides more than 40 cross-platform controls and layouts which are mapped to native controls at runtime,
which means that your user interfaces are fully native.

For example, The login page we create for VitalTrax is one C# script, but will work for any of the
platforms mentioned above. If you want distinct features for any platform, you can simply wrap
that code in a processor direction (#IF statement), and Xamarin.Forms will ignore it on builds for
platforms outside of the #IF statement. 

For example:

```C#
#if WINDOWS_UWP
        public static string HockeyAppId = "";
#elif __ANDROID__
        public static string HockeyAppId = "";
#elif __IOS__
        public static string HockeyAppId = "";
#endif
```


### WebView ###

  ![WebView]({{site.baseurl}}/images/screens/WebView.png)

Xamarin has a built-in WebView solution which allows the developer to access webpages within 
their application by embedding the device’s native browser via a Xamarin control. As a developer, 
you want to keep the user engaged within your application and not ever have to leave, because once 
they are going, it’s difficult to grab their attention again. 

Moreover, with a WebView, the healthcare surveys VitalTrax created for the browser can be reused,
saving them both time and money.

This page was generated purely with C#, and included no XAML. When WebPageView.xaml.cs is initialized, 
the constructor begins building a Label, dubbed the header, in addition to a WebView which has a property
titled source, and allows us to pass in the URL of a clinical trial.

From there, we create buttons to navigate forward and back, and attach event handlers to perform those
operations.

```C#
  Button backButton = new Button()
  {
      Text = "Back",
      HorizontalOptions = LayoutOptions.Start
  };


      backButton.Clicked += delegate
  {
          // Check to see if there is anywhere to go back to
          if (webView.CanGoBack)
          {
              webView.GoBack();
          }
          else
          { // If not, leave the view
              Navigation.PopAsync();
          }
      };
```

Finally, we create a new StackLayout, set its children property to be header, WebView, 
and buttons we created earlier, and set that as the Content element in the view (XAML). 
We now have entire UI generated from nothing more than C#.


```C#
    // Build the page and add it to the XAML
    this.Content = new StackLayout
    {
        Children =
        {
            header,
            webView,
            backButton,
            forwardButton
        }
    };
```

### Landing Page ###
  [Landing Page Android]({{site.baseurl}}/images/screens/LandingPage-Android.png)


An alternate approach to building your UI in C# would be to utilize XAML and the intellisense Xamarin
offers. On pages where the experience will not change radically, or we are sure of how many elements
we will need to draw to a page beforehand, we find that this works very well.

When users first load the VitalTrax application, they are greeted with the Landing Page, which prompts 
users to login. Once authenticated, they can begin to search for trials or continue ones already in progress. 

To handle authentication, we used RestSharp, a C# framework for performing REST operations. The healthcare 
industry has strict regulations, therefore we can’t simply login with a user name and password. Therefore,
we’ll need to make one request for an access token, which we can then pass around on subsequent requests.
This token is essentially our key for accessing patient data, such as trials and a profile. 

We’ve illustrated how that works in [this GitHub repository.](https://github.com/DaveVoyles/Xamarin-Rusable-Components)


### Offline Sync / Local Storage ###

```C#
/*
 * Overview
 * --------
 * Offline sync allows end users to interact with a mobile app--viewing, adding, or modifying data--even
 * when there is no network connection. Changes are stored in a local database. 
 * Once the device is back online, these changes are synced with the remote service.
 * 
 * To add Offline Sync Support:
 *  1) Add the NuGet package Microsoft.Azure.Mobile.Client.SQLiteStore (and dependencies) to all client projects
 *  2) Uncomment the #define OFFLINE_SYNC_ENABLED
 *  
 * Use this package:          https://www.nuget.org/packages/Microsoft.Azure.Mobile.Client.SQLiteStore/
 * Step-by-step instructions: http://go.microsoft.com/fwlink/?LinkId=620342
 * 
 * To initiate a sync in an Android or iOS app:
 *  1) pull down on the items list.
 *   
 */
#define OFFLINE_SYNC_ENABLED
        const string offlineDbPath = @"localstore.db";

        /// <summary>
        /// Creates a new local SQLite database using the MobileServiceSQLiteStore class.
        /// </summary>
        private TodoItemManager()
        {
            this.client = new MobileServiceClient(Constants.ApplicationURL);

#if OFFLINE_SYNC_ENABLED
            // Init local storage. Creates a table in the local store that matches the fields in the provided type. 
            var store = new MobileServiceSQLiteStore(offlineDbPath);
            store.DefineTable<TodoItem>();

            // Initializes the SyncContext using the default IMobileServiceSyncHandler.
            this.client.SyncContext.InitializeAsync(store);

            // Uses the local database for all create, read, update, and delete(CRUD) table operations.
            this.todoTable = client.GetSyncTable<TodoItem>();
#else
            this.todoTable = client.GetTable<TodoItem>();
#endif
```

When users have limited connectivity, VitalTrax still wants them to be able to fill out forms without
the fear of losing their work. With Azure Mobile Applications (formerly Services), we can create a 
backend in either NodeJS or ASP.NET to broker CRUD operations between the mobile device and our database
in the cloud. Azure Mobile Applications gives you this service out of the box. 


Simply spin up a new Mobile Application in the Azure portal, select the backend you’d like to use,
and select a database in your account to connect to. Additionally, the quick start button can generate
a mobile solution on a variety of platforms (Xamarin, iOS, Android, Cordova) for you, or provide a few 
lines of code to add to your existing application and get started immediately. 


Working in tandem with SQLite,  an in-process library that implements a self-contained transactional
SQL database engine, we can temporarily store data on the device, and when network connectivity resumes,
compare our local database values with those stored in the cloud, and then synchronize them. 


### Push Notifications ###
  ![Android push notification]({{site.baseurl}}/images/screens/Android-Push-Notification.png)


Push notifications can be used to alert a user of an event, whether or not the application is running.
There are a number of ways it can be implemented on each platform, and therefore we had several 
evangelists tackle this problem. 

Since Xamarin.Forms uses Support Libraries 23.3.0 you will be restricted to going through the standard
Google Cloud Messaging route.

While setting up push notifications to work with a Xamarin.Forms app, one of the biggest issues we 
consistently ran into was weeding through deprecated technology.  For this reason, it was suggested to our
team to set up Xamarin Android notifications through Google’s system specifically, rather than route
them through Azure Notification hubs. 

These two specific Xamarin docs were the most recently updated and helpful with setting up push 
notifications using Firebase Cloud Messaging (FCM), as opposed to the soon to be retired Google 
Cloud Messaging (GCM).

Setting up the application, we ran into some issues with incompatibility between NuGet packages,
requiring use of older beta versions. In addition, it was difficult to get Google Play services,
which is needed for FCM, installed onto the Visual Studio Emulator via Gapps. Only this guide had a
package that managed to work with a specific emulator. 


### Dynamic UI ###

A dynamic UI is useful for when you want to generate a distinct look for each of your customers,
but also when you do not have a pre-defined number of items to draw on the screen. The Fidelity app 
on iOS and Android are a fantastic example of this, as one individual’s stock selection will like be
different from that of other people. Fidelity’s Xamarin application can generate tables and stock quotes 
for each of notes in your account. 

VitalTrax was looking for a similar solution, where a unique logo, color scheme, and questions could 
utilized for each user and survey. To solve this matter, we created modular components, or widgets, in C#, 
which can generate the XAML, or markup (UI) for our page. 

For example, to create a text Label, we’d write a function like this:


```C#
		  /// <summary>
			/// Empty container to hold a person's attributes: Name, Role, Image, etc.,
			/// </summary>
			public StackLayout personContainer = new StackLayout()
			{
			};
		

			/// <summary>
			/// Creates a Label based on the user's name
			/// </summary>
			/// <returns>Label w/ user's name</returns>
			/// <param name="name">What is the name of this person?</param>
			public Label nameComp(string name)
			{
				Label _nameComp = new Label()
				{
					Text 				    = name,
					HorizontalTextAlignment = TextAlignment.Center,
					TextColor			    = Color.Black,
					Margin				    = new Thickness(0, 20, 0, 5),
					FontSize 				= 18
				};
				return _nameComp;
			}


	  	/// <summary>
			/// Create a person (StackLayout) from local data
			/// </summary>
			/// <returns>The person.</returns>
			private StackLayout createPerson() 
			{ 
				// Create a new component from Component.cs library
				_component   = new Component();
				var myPerson = _component.personContainer;
	

				// Add all of the components needed for a person
				myPerson.Children.Add(_component.nameComp(_component._name	   ));
				myPerson.Children.Add(_component.roleComp(_component._role	   ));
				myPerson.Children.Add(_component.photoComp(_component._imageUri));
	

				return myPerson;
			}
```

C# has this concept of default parameters, where a function will use a pre-defined value if we do not 
pass one in as an argument. We used these for many functions, so that the various elements of UI would 
have always have a template to begin with. To change these values, we can make a REST request to 
VitalTrax’s server and pull down a JSON file which defines the font style, size, any images, or questions
to place on a questionnaire. 

### DevOps ###
  ![Kevin Dev Ops Overview]({{site.baseurl}}/images/8.png)

On day one of our visit, and wrapping up on day two, Kevin Remde took the lead on the DevOps approach,
drawing out not only VitalTrax’s entire product lifecycle, but also illustrating where key improvements
could be made by considering several DevOps best practices. Although VitalTrax already had an in-depth 
solution in place, and were already using Visual Studio Team Services (VSTS) for Continuous Integration
(CI) and Continuous Deployment (CD), Kevin still found areas for improvement. 

TODO: Insert dev ops diagram
  ![Dev Ops Diagram]({{site.baseurl}}/screens/VitalTrax-VSM.png)


**The Findings**

VitalTrax is comfortable with a 3 month product development lifecycle for new versions of both their 
Wing Enterprise and new Wing application.  They made it clear that their customers in the pharmaceutical
industry are more comfortable with and would prefer fewer frequent updates.  In estimates of the 
lead time taken for each process in their product lifecycle, it was estimated that the full time 
for a new feature is approximately 34.5 days (6 weeks).  VitalTrax leadership were not surprised by this.
They try to do two or three of these sprints for each quarterly release of their products.

The value stream map shows six main processes that make up their product development lifecycle:
1.	Iterative Planning
2.	Local 
3.	Dev
4.	QA (Quality Assurance)
5.	UAT (User Acceptance Testing)
6.	Production


**Iterative Planning** – This is where the product invention, features, and fixes are defined and prototyped.
Because this process is iterative by design, there were no estimates given to how long each specific
process would take.  The end of the process involves the planning of springs, and the development of mock-ups.
It was discovered that although the goal is to have mock-ups done before development starts, it often starts 
sooner.  One person is responsible for doing all of the mock-up work, and problems sometimes arise when
developers start work based on partially-done mock-ups.


**Local** – this is where the developers work on product on their local workstations.  It was determined 
that at some point they want to include more local unit and/or automated testing during this stage, to 
potentially reduce the scrap-rates – the percentage of times the code has to be handed back after it 
reaches the Dev and QA smoke testing.

**Dev** – VitalTrax has a “Dev” environment that hosts the application versions ready for testing.  
Developers, who work from a git fork off a “dev” branch do a pull request; features going into stories
and stories making up the release.  Code is reviewed before the pull request is approved and code is merged. 
All features and stories for a release must be complete and approved before moving to this Dev step. 
Once approved, VitalTrax has implemented Continuous Integration and Continuous Deployment in VSTS.

  ![Dev Ops Build Def]({{site.baseurl}}/screens/VSTS-Build-Definition.png)
  ![Dev Ops Release Def]({{site.baseurl}}/screens/VSTS-Release-Definition.png)


Currently, these Dev, QA, and UAT environments are hosted on virtual machines in Azure.  
They are built and configured using PowerShell scripts, but are not re-built with every new product
release lifecycle.  

Todd Kueny on Continuous Integration, Continuous Deployment and Release Management in VSTS: 

 > “All I can say is, I love this process.  This makes it so much simpler and so much easier to understand.. 
 > So much easier 1) to deploy and 2) to know what’s in the deployment.  You know, I’ve done this a lot at
 > a lot of different companies, and this is the best process I’ve had by far, because it
 > provide the complete traceability and it’s like just a one-click thing.”

Todd Kueny on Visual Studio Team Services:

> “There’s just so many cool things you can do with this. I don’t even know how to explain it.  
> This is just one of the coolest tools I’ve used in a long time.”


Isaac Collier on moving to ARM template deployments (IaC):

> “I had started to look into using the resource manager templates instead. It was a little bit 
> overwhelming at first just because you look at ‘em and it’s just like pages and pages of text.  
> So figuring out what I need from those is still something that needs to be done.  But I would like
> to switch to that because there are some pieces right now that I’m doing manually that could certainly 
> be automated.  


> The other goal from my perspective is around looking into processes for being able to continually do 
> this because what we’d like to be able to do is to continue to iterate on our infrastructure, and 
> continue to improve it.  And make changes in a way that… I’d like to do this in less of a manual way.  
> If there’s a way that we can make changes in a more structured way; meaning that we’re changing some
>  code or a template or something like that and then redeploying it, or the changes being deployed 
> through a script of some kind, I’d prefer to do that.”

  ![Azure-Resource-Groups]({{site.baseurl}}/screens/Azure-Resource-Groups.png)


One of the results of the VSM was a noted desire to configure these environments using Azure Resource
Manager templates, to add configuration management functionality in the form of Azure Automation Desired
State Configuration (DSC) as well as virtual machine performance scaling using Azure Scale Sets.

**QA** – a “Test” environment (virtual machines in Azure) is where the product goes through additional
smoke tests and validation.  This testing is done by a small number of people, and does involve a small
amount of scripting.  It was determined that VitalTrax would like to add more automation to their testing
both in the Dev and QA stages of their product lifecycle, as there is currently too much manual effort here.  

**UAT** – This is where the customer has some involvement.  Earlier in the process (ideally during the
iterative planning) the customer will contribute to the plan, and these contributions will be tested both
by VitalTrax and by the customer during this stage.  It was noted that there is often not enough of this 
contribution early-on, and so too often the customer is not always seeing what they expected, causing
re-work to be done.  It was difficult for VitalTrax to estimate a scrap rate here, as they’ve only gone
through this once; and it wasn’t a particularly pleasant experience. One item for future consideration
was to use HockeyApp to support the UAT of their new mobile application.

Once UAT is complete the signoff takes place and the code is ready for production.

**Production** – The leadership team does a sign-off on the completed release, and then the Wing 
Enterprise hosted code is updated, and/or the new Wing mobile application version is published to the
applicable app store.

In day 2 and through day 3, while the rest of the team focused on the Wing mobile app itself,
Kevin lead Isaac in some introduction to and work with Infrastructure as Code (IaC) in the form
of Azure Resource Manager templates.

Isaac and Kevin came up with four areas to address; some while they were together and others for
future investigation. 

Below is the list, along with links to related resources:

1.	Ongoing – source/change control
2.	[High Availability](https://github.com/KevinRemde/VTestDSCSS ) (Load Balancing and Scale Sets) 
3.	[SQL Server Scale](https://github.com/Azure/azure-quickstart-templates/tree/master/sql-server-2014-alwayson-existing-vnet-and-ad) (database storage) 
4.	[Set up VMs with encrypted VHDs](https://docs.microsoft.com/en-us/azure/security/azure-security-disk-encryption) 


As Isaac worked on getting a basic template deployment working, Kevin implemented a proof-of-concept
template (#2 on the list above) that quickly deploys an Azure Scale Set of virtual machines. 
Additionally the sample demonstrations the automated creation of an Azure Automation service, and 
deploys a sample web server configuration to the members of the scale set using Azure Automation 
Desired State Configuration (DSC).


## Conclusion ##

This section will briefly summarize the technical story with the following details included:

With a mobile application, VitalTrax now has the opportunity to embed HTML5 based questionnaires, or create them 
natively with C#, thereby offering greater flexibility than ever before. Moreover, each company can now have a 
distinct look and feel, with a customized color scheme, font, and logo. 

Perhaps one of the greatest benefits however, is that users can now fill out a questionnaire despite not having 
internet connectivity, due to the implementation of offline sync. Building on that, with push notifications, 
users can be notified when a trial is due, set to expire, or a new one arrives.

These are all key factors which allow VitalTrax's Wing Enterprise application to stand out from the competition
and break through an industry not typically known for adopting the latest technical feats.  

### General lessons ###

Push notifications proved to be as difficult as anticipated. Xamarin has the aims to combine three mobile 
platforms into one, and at times each of those platforms can have distinct ways of implementing features. Push
notifications is one of those features. 

Additionally, because the mobile world moves to quickly, documentation becomes out of date soon after it is created,
thereby creating a bit of a challenge around finding tutorials or NuGet packages which were still relevent. 

In terms of reusable code, the Xamarin Reusable Components GitHub repository we created offers a lesson in not
only how to authenticate against a RESTful API with RESTsharp, but also has a sample on how to generate XAML
from C# code. This is particuarly useful when the developer is unsure of how many elements will need
to be drawn on a page. 

### Opportunities going forward: ###

Going forward, VitalTrax would like to flesh the Xamarin.Forms application to include push notifications for 
iOS. Additionally, they would like to have healthcare companies create a questionnaire in their web portal backend, 
which would generate a JSON file with those questions, and dynamically generate a form in C# for the Xamarin application
when the end user logs in to fill out a survey. 

> “Microsoft won the technology battle fair and square. We looked at all of the options available to us,
> and they simply offered the best.”

 **– Syed Zikria, Founder/CEO**


## Additional resources ##

### GitHub Repositorites ###

- [Xamarin Reusable Components](https://github.com/DaveVoyles/Xamarin-Rusable-Components)
- [VitalTrax's repository for this project]https://github.com/DaveVoyles/xam-test-andy()

### Useful Xamarin Documentation ###

- [SQLite for Xamarin.Forms](http://go.microsoft.com/fwlink/?LinkId=620342)

### Misc ###

- [Zeplin.io](http://www.Zeplin.io) - Useful for mocking up UI on mobile platforms
- [Xamarin app from Evolve conference]( https://github.com/xamarinhq/app-evolve)