---
layout: post
title: "Finding missing kids by harnessing Azure Functions"
author: "Anthony Bartolo"
author-link: "https://twitter.com/WirelessLife"
#author-image: "{{ site.baseurl }}/images/authors/photo.jpg"
date: 2017-01-31
categories: [DevOps, Azure App Service, Mobile Application Development with Xamarin]
color: "blue"
#image: "{{ site.baseurl }}/images/imagename.png" #should be ~350px tall
excerpt: Alongside members of the City of Calgary and Calgary Police, Missing Children Society of Canada partnered with Microsoft in an innovation workshop to understand how Azure Functions, Xamarin, and DevOps best practices can better address their need to quickly provide police with information about children who are reported missing. 
language: [English]
verticals: [Public Safety]
---

Children today are technology natives, harnessing technology to play, share, and learn. With this empowerment comes responsibility, both  of parents and of the children themselves. Who kids chat with online and what they share can come at a price should it fall into the wrong hands.  This challenge is something that keep CEO Amanda Pick and the team at Missing Children Society of Canada (MCSC) up at night as more and more predators are using online sources to lure children.

Last year 45,000 children in Canada went missing. The sole mission of Calgary-based non-profit MCSC is to bring every child home. Part of their challenge is acquiring information on a child (such as last-known whereabouts and contacts) and providing that information to local law enforcement.

Alongside members of the City of Calgary and Calgary Police, MCSC partnered with Microsoft in an innovation workshop to understand how Azure Functions, Xamarin, and DevOps best practices can better address their need to quickly provide police with information about children who are reported missing. The resulting application used the following services and practices:

- Azure Functions
- Azure Storage
- Continuous Integration
- Infrastructure as Code
- Visual Studio Team Services
- Xamarin Tools
 
The goal of the 4-day project was to enable MCSC to use automated Azure functionality to extract last-known coordinates, who the child was in contact with, and the sentiment of conversation from social-media platforms such as Twitter and Facebook. The innovation workshop team comprised members from Microsoft, MCSC, Calgary Police, and the City of Calgary and included the following:

- Tricia Bailey – Chief Operating Officer at MCSC
- Mike Williamson – Digital Forensic Examiner at Calgary Police Service
- Abdel Mohamed – Senior Programmer Analyst at the City of Calgary
- Jef King – Sr. Technical Evangelist (Azure Functions Lead)
- Pierre Roman – Sr. Technical Evangelist (DevOps Lead)
- Mickey MacDonald – Sr. Technical Evangelist (Azure Functions support / Xamarin Support)
- Mark Arteaga – MVP / RedBit Development President (Xamarin Lead)
- Sage Franch – Technical Evangelist (Xamarin Support)
- Adarsha Datta – Technical Evangelist (Xamarin Support)

To support youth helping youth via tech, the team also included representatives from the Silicon Halton Tech Under 20 user group and a Microsoft Student partner. 

<img alt="Photo montage: team members; sign, badge, and logo of Calgary Police Service; logos of MCSC, City of Calgary, and Microsoft" src="{{ site.baseurl }}/images/MissingChildrenSocietyofCanada/image1.png" width="960">

## Customer profile ##
For 30 years, [Missing Children Society of Canada](http://mcsc.ca/) has provided help and support for families with missing children. Their programs serve families and law enforcement in the critical hours after a child goes missing. MCSC has three main programs of service: Investigations, Family Support, and the Search Program.

<iframe src="https://channel9.msdn.com/Series/Customer-Solutions-Workshops/Enlisting-Microsoft-Technologies-to-Aid-Missing-Childrens-Society-of-Canada-to-Find-Lost-Children/player" width="960" height="540" allowFullScreen frameBorder="0"></iframe>
 
## Problem statement ##

In meeting with Amanda and the team at MCSC, we learned challenges in acquiring information when a child is reported missing. Aside from the immediate information about the child, such as what the child was wearing and where he or she was last seen, acquiring additional information sometimes took days or possibly weeks of research.

The team at Microsoft Canada brainstormed a way to help quicken the research process. Our goal was to automate the research conducted online by extracting data directly from the missing child’s social-media account. 

<img alt="Whiteboard with notes and diagram" src="{{ site.baseurl }}/images/MissingChildrenSocietyofCanada/image2.jpg" width="720">

The first half day was spent establishing mapping of the current delivery of information to police after a child is abducted. 

<img alt="High-level flowchart of entire process" src="{{ site.baseurl }}/images/MissingChildrenSocietyofCanada/image3.png" width="840">

The plan established was as follows:

1. Enable parents and/or children to register their social-media accounts with MCSC.
2. When a registered child is reported missing, the child or parent could invoke the Child Finder solution (enabled via Azure Functions) to capture the last-known whereabouts of the child, who the child was in contact with, and the sentiment of the conversation by including the hashtag #hfm (“help find me”) on their social-media feed.
3. The Child Finder solution would capture the required information directly from the API of the social-media provider.
4. The Azure Function would send the captured information directly to law enforcement, notifying them that a child had been reported missing in their area and providing additional information acquired via social media.
5. Law enforcement would log in to the application via Azure Active Directory Authentication to ensure all data captured was accessible only by those who had been assigned access.
6. Law enforcement would be able to see the reports and additional information on the children reported missing.

**NOTE:** The #hfm hashtag can be used only by those who have access to the user name and password for the account. This restriction respects the child’s privacy and empowers only them or people they trust (such as their parents) to trigger the Azure Function to monitor their social account. 

The value stream mapping exercise led MSCS and Calgary Police to realize that its current way of delivering information needed to be modernized to ensure quicker transport. The exercise proved valuable even though this solution comprised a net-new process. Conducting the value stream mapping exercise provided a resource map for the team to follow.

![Dataflow diagram of entire process]({{ site.baseurl }}/images/MissingChildrenSocietyofCanada/image4.png)

Next, our team mapped the data flow of the proposed project.  Security and self-empowerment were top priorities because of the sensitivity of the information captured and delivered. The Twitter OAuth enablement was established to empower the child and/or parent to enable monitoring of an account. The #hfm hashtag empowered the child and/or parent to enable information to be captured from their account. An officer login screen was included to ensure that information captured was viewed only by law enforcement.
 
## Project objectives ##

The project on behalf of MCSC included the following:

Azure Web App

- A portal on Azure was created for parents or kids to register social-media accounts to be monitored.
- Azure Functions were used to monitor registered social-media accounts when the trigger hashtag is detected. (Azure Functions allow small pieces of code to be run in the cloud. Code can be created and run for the problem at hand without requiring infrastructure or even a whole application to run it.)
- When the hashtag is detected, all metadata (such as location, time, and date) and sentiment is captured and made available to law enforcement through the corresponding Xamarin mobile app


Xamarin mobile app

- Authentication rules were created to ensure that only authorized personnel gain access.
- Law-enforcement personnel are notified of new reports and can access metadata and sentiment through the app.
- MCSC staff are also notified via the same app so that they can aid law enforcement.

DevOps practices

- Value stream mapping was utilized to capture the process flow and the collection and dispersion of database information through the Azure Web App and the Xamarin mobile app.
- GitHub repositories for infrastructure (Azure Functions), authorization (Azure Web Portal), and client (Xamarin) were used throughout the project.
- Continuous integration streams were created to facilitate both the build and deployment.
- Infrastructure as Code methodology was used to deploy the environment and the other components to ensure consistency.


## Solutions, steps, and delivery ##

### The process ###

The child (or a parent on behalf of the child) registers a social-media account via the [Missing Children Society of Canada web page](http://mcsc-authorization-dev.azurewebsites.net/), ensuring that no one is tracked unless he or she wants to be. This was developed as a Node.js application named “authorization” and is currently connected to Twitter.

<img alt="Flowchart of components" src="{{ site.baseurl }}/images/MissingChildrenSocietyofCanada/image5.png" width="840">

Alongside this is an Azure Logic App that listens for tweets that contain the hashtag #hfm. When the app finds one, it adds the tweet to a Service Bus queue.

When a message appears in this queue, an Azure Function picks it up and checks the authorization repository in DocumentDB. If the user who made the tweet is registered, the message is added to the next queue; if user is not registered, the message is ignored.

From the next queue, another Azure Function picks up the message and augments the data. Currently, the process adds the location as reported by the device that posted the tweet or deduced from the IP address of the tweet’s origin). The augmented message is added to a third queue.

The last stage in the process is an Azure Function that stores the message in an appropriate format in DocumentDB.

After the Azure Function processes are complete, the augmented data is exposed for client use through a REST API. The API provides access to the user-account information and the captured social-media data. This API is implemented in Node.js and is running in an Azure Web App instance. This same API is also responsible for authentication within the client application.

For the finders/police officers, a Xamarin mobile app connects to the REST API and enables the finder or police officer to visualize the data and even track location (if location data is available). To use this application, a finder or officer must register and log in. In a future release, reporting of all children who go missing will be published to a centralized dashboard that will forward the information to law enforcement within a specified area.

### How this was accomplished ###

The following technologies were used:

**GitHub** to store all source code, which is released under the Apache license. By making the source code publicly available, we can actively take contributions and enable the wider community to use a system like this to aid in the finding of missing persons or in other areas where people need to self-identify and be tracked.

**Visual Studio Team Services** to support continuous integration builds and the deployment pipeline. With the project team in both Toronto and Calgary, utilization of VSTS also enabled teams to share code and track work from either city.

**Azure** to provide the simplest, cheapest, and most scalable services. Given the current set of processes and storage requirements, the solution should be able to handle huge volumes of data with very little latency.

A simple Node.js app captures the self-registration of users who want the hashtag enabled. This page can be styled to match the needs of the client.

The data-ingestion pipeline ensures rapid finding of content that a user has self-registered. In the current version, we chose to use a Logic App because it costs much less than a long-running Azure Function. The Logic App queries Twitter every 30 seconds. When it finds a tweet that contains the specified hashtag, it adds the tweet to the processing queue.

The pipeline from there is built with a set of robust Azure Functions and queues that ensure proper handling even if failures occur. The Azure Functions also enable massive scale while minimizing cost. Each step of the pipeline has a specific task: First, determine whether the person who tweeted is registered. If not, the message is ignored; no copies are held anywhere to ensure privacy. The next step is to pull in additional information. In the current version, we focused on adding geospatial data; however, other data could also be included. Last, the item is pulled off the final queue and placed in DocumentDB, which is then accessible through a REST API to the Xamarin client.

**Xamarin.Forms** to allow deployment to Android, iOS, and Windows with a consistent user experience across all devices, and to provide a streamlined development process. The client app provides the captured data in an easy-to-understand format. Using the strong REST support provided by Xamarin, we connect to the REST API, import the data, and display that data in a list-style format. In the client app, the user can navigate through the data (or “cases”) and sort by Open, Archived, and Watched. When a specific case is selected and location data is available, the client application provides a map view of the last-known location of the social-media user. It is worth noting that this application uses the MVVM architecture pattern-–the standard architecture used by RedBit for their own mobile apps. With guidance from the RedBit developer team, some components of the MVVM Light source code were used.

![Whiteboards with details of Xamarin mobile app]({{ site.baseurl }}/images/MissingChildrenSocietyofCanada/image6.jpg)

### Moving forward ###

**Facebook/Instagram Logic App**: The system is built to enable multiple channels from social networks. These networks would feed the same pipeline and have their own characteristics.

**Twitter Logic App**: Currently, the Logic App is using the JefKingTweets account as its point of contact; it will be moved to the MCSC account for production.
 
**Logic App tweet-processing interval**: The interval is currently set to 30+ seconds to streamline data capture; it can be increased or decreased as required.

**Data-augmentation Azure Function**: Additional context can be added to the system to provide more insight and context to finders. For example, we could add tweets from the user’s history to provide clues about friends or favorite places; watch for more tweets in the next 48 hours and alert finders if a location is identified or changed; and gather other geo-related tags that could give finders clues about the missing child’s location, if the actual location is not available.
 
**Automated Azure Function configuration**: Azure Function configuration was done manually during the project; in production, this should be automated and driven through Visual Studio Team Services for easier management and better security.

### DevOps practices ###

The following sections highlight the DevOps practices implemented during this project.

#### Value stream mapping ####
This exercise is typically conducted against an existing set of processes and activities tied to the development of a specific solution. In this case, we examined current process and received suggestions from MCSC, Calgary Police, and the City of Calgary about what could be improved. As seen earlier, both the Azure Functions and the client application were mapped and vetted by the team. Improvements were identified, implemented by the assigned team of developers, and then introduced to streamline the development of the solution while providing a strong automation component. The following two suggestions were added to this project as a result of the value stream mapping exercise:

- Mobile app authentication: Enabled to provide police secure access to the missing-child reports provided by the prescribed Azure Function as captured from Twitter
- Twitter profile image: Added to the report in hopes that the missing child’s profile image would be displayed for identification

#### Code repository ####

A new GitHub organization was created and multiple repositories were added to facilitate multiple teams working on the project. The org can be found at [https://github.com/CDN-Missing-Children-Hack](https://github.com/CDN-Missing-Children-Hack)

<img alt="Screen shot of CDN-Missing-Children-Hack repository on GitHub" src="{{ site.baseurl }}/images/MissingChildrenSocietyofCanada/image6.png" width="720">

The repos added to the org are as follows:

- authorization: Client authorization for social media
- client: Xamarin app development
- client-api: API for client devices
- client-registration: Push-notification registration
- infrastructure: Azure infrastructure
- messaging: Back-end message passing and processing


#### Continuous integration ####

We implemented four continuous-integration streams via Visual Studio Team Services to facilitate build and deployment:

- client-api-ci
- messaging-ci
- authorization-ci
- Infra-DEV-CI

<img alt="Screen shot of build definitions in Visual Studio Team Services" src="{{ site.baseurl }}/images/MissingChildrenSocietyofCanada/image7.png" width="960">

Each of these build definitions include the tasks to generate proper builds, including npm, gulp, Grunt, and others based on language and build type. The Xamarin client apps were deployed via HockeyApp from Visual Studio Team Services.

#### Continuous deployment ####

Four release-management queues were created to allow the team to produce software in short cycles, ensuring that the software can be reliably released at any time. It aims to build, test, and release software faster and more frequently. HockeyApp provided additional flexibility by delivering apps directly to devices without the need of submitting the client app to a mobile store. The approach helps reduce the cost, time, and risk of delivering changes by allowing for more incremental updates to applications in production. 

<img alt="Screen shot of release definitions in Visual Studio Team Services" src="{{ site.baseurl }}/images/MissingChildrenSocietyofCanada/image8.png" width="720">

#### Infrastructure as Code ####

The infrastructure for the solution was created by using a series of JSON templates that are used to deploy the environment and the other components that the solution requires. The JSON templates are checked in to the GitHub infrastructure repo and are subject to the same build and release management process as the rest of the solution code. Following the process provides consistency of deployment in a development, testing, or production resource group.

<img alt="Screen shot of infrastructure release definition in Visual Studio Team Services" src="{{ site.baseurl }}/images/MissingChildrenSocietyofCanada/image9.png" width="960">
 
## Conclusion ##

The team successfully developed the registration for self-identification, the data-ingestion pipeline for processing and augmenting the data, the API for the client devices, and the client application for their devices.

With these components in place, the MCSC team can now start building out and improving the services rapidly. Because all features were built alongside a DevOps pipeline, improving and adding more value is much more streamlined.

<img alt="Screen shots of mobile app" src="{{ site.baseurl }}/images/MissingChildrenSocietyofCanada/image10.png" width="960">

In addition, during the innovation workshop the team successfully extracted GPS data from Twitter and Facebook and presented that data in the Xamarin application.
 
The mapping portion of the project highlighted the need for a dashboard to be viewed by a central office prior to sharing the information with local law enforcement. This work is currently being planned for the next iteration of this solution.

MCSC, Calgary Police, and the City of Calgary recognized the value of continuously improving code and delivering incremental new features through GitHub and HockeyApp distribution to mobile devices.

