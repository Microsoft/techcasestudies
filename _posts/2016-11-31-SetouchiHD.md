---
layout: post
title:  "Learning from a Microservices/Mobile Hackfest with Setouchi Holdings"
author: "Tsuyoshi Ushio"
author-link: "#"
#author-image: "{{ site.baseurl }}/images/authors/photo.jpg"
date:   2016-12-05
categories: [DevOps]
color: "blue"
#image: "{{ site.baseurl }}/images/imagename.png" #should be ~350px tall
excerpt: Add a short description of what this article is about.
language: English
verticals: Manufacturing & Transportation
---

Setouchi Holdings is a company which have a several services related small airplane manifacturing. They are developing a new reservation system
for small airplain for sight seeing for Setouchi alrea. They work with Ci&T and Creatinoline using Azure. They are using the latest lean/kanban 
approach conbined with Azure technologies. However, they are new to Azure, they enjoyed the Mobile/Microservices DevOps hackfest with Microsoft. 

Begin with an intro statement with the following details:

- Solution overview: 
  Small airplane reservation system. They already have a good leadtime for web deployment. However, they also want to have blue green deployment
  and automated mobile CI/CD pipeline.

  ![Architecture]({{site.baseurl}}/images/2016-11-31-SetouchiHD/architecture.jpg)

- Key technologies used:
  Azure Web/Api apps with C# (Infrastructure as Code) 
  Visual Studio Team Services (CI/CD/Release Management)
  Swift(iOS), Testflight, and Fastlane (Mobile DevOps)
  Goal integration with VSTS (Telemetry)

  ![Hackfest Members]({{site.baseurl}}/images/2016-11-31-SetouchiHD/allmembers.jpg)  

- Core Team: 
  Geovanne Borges Bertonha @geobertonha - Software Architect/DevOps
  Andre Ogura Dantas  - Software Architect/ iOS Specialist
  Leandro de Lima Machado - Software Engineer
  Andre Santos Kano - @drekano - Software Engineer
  Tadahiro Yasuda @yasudatadahiro - Creationline
  Daisuke Ono - Creationline
  Tsuyoshi Ushio @sandayuu - Technical Evangelist - DevOPs
  Naoki (NEO) Sato @satonaoki - Senior Technical Evangelist - Azure General

## Customer profile ##

- [Setouchi Holdings](http://setouchi-hd.com/)
- Manifacturing for small airplane and transportation services
- Hiroshima, Japan. (Hackfest had in Tokyo, Japan)
- The reservation system for small airplane rental.

They have two partners for this project. 
Development: [CI&T](http://www.ciandt.com/home) 
Infrasturcture: [Creationline](http://www.creationline.com/en/)

 
## Problem statement ##

They have a great knowledge of Agile and enugh technical skills. However, they were new to Azure and VSTS. Also, they also had a lot of manual 
process. This is partly because they had lean approach. They have an enough knowedge to automate it however, they intentionally didn't do it. 
Because this is a new project. Unless really need it, they didn't automate it. I think this is the right approach. However, Mobile pipeline 
automation with VSTS and Blue green deployment on Azure are biggest issue for them. 
 Also, they want to integrate Goal, which is CI&T internal dashboard system, with VSTS. They are usually use Jira. They like VSTS kanban system,
 so that they wanted to integrate VSTS with Goal. 

## Solution, steps, and delivery ##

We solved these problem by follwing steps. 

### 1. Value Stream Mapping 

We started with Value Stream Mapping. However, it is not easy. Beause this project is just started. They have strong lean principle. That is why
they didn't predict a lot in advance. When we had a value stream mapping, they just finish, a few sprints. They had a scoping strategy for 
Microservices. They gradually grow their development scope. They started with Web application development, then mobile, then Microservices... and so on.
We couldn't see whole picture at that time. However, we discussed thier development process of the last time, and discuss with them. 

![Value Stream Mapping Discussion]({{site.baseurl}}/images/2016-11-31-SetouchiHD/VSMdiscussion.jpg)  

![Value Stream Mapping Discussion]({{site.baseurl}}/images/2016-11-31-SetouchiHD/ValueStreamMapping.png)  


The leadtime of the web site is not bad. 2days. My Recommendation is Release Management. However, they already good at the leadtime, I also recommend 
feature flags.

However, agile is about "Embrace change", right? When we tried the hackfest, their request is slight different. Release Management is the same.
But they had a problem of the Mobile CI/CD pipeline. They need automation. Also, They need to blue green deployment for the new Microservices, which 
is an Api apps. Also, they were going to have load testing for the slot. It will cause the "Noizy neighbor" problem, which means if you had a load testing
for a slot, it will effect the production slot as well. 
 They don't familier with VSTS, so they wanted to know about how to organize Kanban and automated "Retain indefinately" flag for a release.

Let's get hacking!

### 2. Blue Green Deployment without Noizy Neighbour problem

We started discussing about the blue green deployment. The blue green deployment itself is very easy for us. Azure Web/Api apps already have the deployment
slot and swap functionality. It is really fantastic feature. Also, the Visual Studio Team Services supports deployment and swap functionality by the tasks.
We can easy to configure these pipeline. 

![Swap deployment slots]({{site.baseurl}}/images/2016-11-31-SetouchiHD/deploymentslot.jpg)  


 However, the problem is the noizy neighbour problem. Naoki (NEO) Sato toled them a solution for this problem. We solve this for adding an App Service Plan.
 Then change the App Service Plan of a slot. When you use Web/Api apps, the resources are shared among the slots. However, if you change only one slot, it 
 means these slots have different App Service Plan. Then Web/Api apps, allocate different resources among the slots. 
Using this technique, you can avoid noizy neighbour problem. 

  ![App Service Plan]({{site.baseurl}}/images/2016-11-31-SetouchiHD/AppServicePlan.jpg)

As a practice of Infrastructure as Code, we use the ARM template after finishing configure resource via Azure Portal. Before deploy it, we can download
the template and deployment script. It is totally easy way to achieve Infrastructure as Code on Azure.



### 3. Automated Retain indefinitely flag 

We wanted to automate whole process. However, they couldn't achieve only one thing for web deployment. After successfully released, they need to click
`Retain idefinitely` flag to keep the successfully deployed artifact. 

  ![Enable Retain idefinitely]({{site.baseurl}}/images/2016-11-31-SetouchiHD/RetainIndifinitely.jpg)

  We have the VSTS REST-API for this purpose. Just update the status of a release. The best way to solve this problem is, using `System.AccessToken` variables to 
access the VSTS REST-API. Some uservoice comment said that it is OK for release management. However, it didn't work. Insted, we use Personal Access Token, for 
this purpose. In the near future, I believe, the VSTS supports the OAuth Access Token. 
  Please create a bat file in your environment(If it is hosted build).

```
curl --request PATCH -H "Accept: application/json" -H "Content-type: application/json"  -d '{ "keepForever": true }' -u %USER_NAME%:%PERSONAL_ACCESS_TOKEN% %SYSTEM_TEAMFOUNDATIONSERVERURI%DefaultCollection/%SYSTEM_TEAMPROJECT%/_apis/release/releases/%RELEASE_RELEASEID%?api-version=3.1-preview.4
```

  Then, you need to set the variables of your personal access token on your Release definition.

  ![Personal Access Token]({{site.baseurl}}/images/2016-11-31-SetouchiHD/PersonalAccessToken.jpg)

Finally, you can use Run Script task and run the bat file. If you use Linux or Mac for build machine, you can use same technique for shell script task.  

  ![Run Script]({{site.baseurl}}/images/2016-11-31-SetouchiHD/RunScript.jpg)

### 4. iOS deployment pipeline automation

They have a lot of problems about the iOS deployment pipeline. The first problem is, they need to build three times for their environemnt. You need to 
include every artifacts into an ipa file. We need to change the connection string for the database. We don't have any measure to avoid to build
three times. Generally speaking, you need to build one time, then you need to pass the artifacts into the Release. For this reason, they only had one 
build pipeline.

  ![Initial Build Pipeline]({{site.baseurl}}/images/2016-11-31-SetouchiHD/InitialBuildPipeline.png)

We improve this step by step. First of all, we build only for dev evnironment on the Build definition as CI. When you change the Git repository, the VSTS
fire the Build definition. We don't want to build ipas everytime for QA/Staging/Production environment. Then we separate the deployment process into Release
Management. We also add the source code into the drop folder, which is shared directory which contains artifacts, then pass the release management and deploy.
However, the problem is the upload time. Especially ipa is very big. It took a long time to upload it. 
 That is why, we remove some unsused files and zip the artifact before upload. It works well.

 Also, sometimes, we had a problem to restore pod files. When we used `pod update` command, sometimes it causes timeout to restore the pod. Once we use `pod install` command instead,
 the time is become within 14 second. 

  ![Improved Build Pipelline]({{site.baseurl}}/images/2016-11-31-SetouchiHD/ImprovedBuildPipeline.png)

Then we create some release definition to adopt this strategy. We need to re-build/test on the release pipeline. It is agains the rule of continuous delivery. 
However, it is better solution for the ipa build. Because we already finish pod restore one the build definition. And we share it via drop folder. 

   ![iOS Release Pipeline]({{site.baseurl}}/images/2016-11-31-SetouchiHD/ReleaseiOS.jpg)

The next problem is the automated deployment for Test flight and automated submit into the Apple Store. They are going to use the [Hockey Apps](https://hockeyapp.net/) for Android Deployment for testing. 
Currently, they use TestFlight. We need to try Test Flight deployment. We use the [fastlane](https://fastlane.tools/), which helps us to automate iOS/Android deployment and release.
We are using VSTS build agent for Mac. They use their own Mac. In the near future, they are moving on MacInCloud.  
  Once we create the Fast file we can automatically deploy into the Test Flight and HockeyApps also submit into the Apple Store. It is very conveinent.

```
fastlane_version "1.109.0"

generated_fastfile_id "{someid}"

default_platform :ios

before_all do
    ENV["SLACK_URL"] = "{Webhook URL from Slack}}" # Webhook URL created in Slack
    ENV["FL_COCOAPODS_PODFILE"] = "SKY TREK/"
    ENV["PILOT_USERNAME"] = "{Your e-mail address}"
  end

lane :beta do
  # build your iOS app
  cocoapods
  
  gym(
    workspace: 'Sky Trek/Sky Trek.xcworkspace',
    scheme: "Sky Trek Dev"
  )

  testflight(changelog: "Testing Fastlane build support")

  slack(
        message: "Successfully distributed a new beta build for testing.",
        success: true
      )

end
```

Also, we need to change the Firstfile depend on the environment. We create one more artifact directory, and put three Fastfiles. After accepting these on the 
Release, we just copy it to Fastfile, according to the environment. Now we were ready to deploy automatically!

  ![Fastlane file structure]({{site.baseurl}}/images/2016-11-31-SetouchiHD/Fastlanefiles.jpg)


Then the slack told us it is successful. The Fast file dosen't inlcude submit into the Apple Store, however, it is very easy. The [Apple Store task](https://marketplace.visualstudio.com/items?itemName=ms-vsclient.app-store)
 uses fastlane inside the task. So we decided to keep on using fastlane to submit the application. After release, we can see the slack notification 
 from the trigger of TestFlight. We've done!

### 6. Kanban customization with Test & Feedback

We need to send some kanban telemetry to the Goal, which is CI&T dashboad system. The Goal is very impressive. It collects a lot of telemetry via Jira. And they 
can get Business Complexity depend on the story. Generally speaking, we have no way to measure the productivity across the company among agile projects. However, CI&T 
makes it enable using the business complexity. Goal calculate it and visualize it. Also it include simple and useful telemetries. 
 Once they start to sell this dashboard, it is very impactful to the world. It only support Jira. However, we want to use VSTS this time.
I recommend them to create a adopter for VSTS. Then, they can supoort two environments.

Now, we start hackhing this. First of all, we need to customize the kanban board column. Then we need to develop the adopter using VSTS REST-API. We need to 
wait the specification of the Goal team. We start hacking the two technical element. We already hacked the REST-API, so we focus on the customize the kanban 
board. It was also very easy. You can not customize a product backlog of the Scrum template, however, you can copy the Scrum template and customize every work
items like a product backlog. 

  ![Goal]({{site.baseurl}}/images/2016-11-31-SetouchiHD/Goal.png)


Once we've got the specification, we going to start the second hackfest!

Also, they are very pleased when I introduce the "Test & Feedback" feature on the Marketplace. This is one of the faborite features of the VSTS for me.


# Conclusion

We automated a lot. Web/Api Swap, CI/CD pipeline for Web/Api apps, and iOS CI/CD pipeline using Fastlane. They can measure the Buiness outcome using the
Goal system. After the hackfest the business outcome increse ...% and also, they can enable 10-deploys-per-day environment on their Web/Api apps.
It is just two days hacking among our team. Most importantly, we were really enjoyed this hackfests.  

## Source code

## Additional resources

